---
layout: post
status: publish
published: true
title: UPS monitoring in Proxmox
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2026-05-18 09:51:49 +0200'
tags:
- proxmox
- ups
- monitoring
- nut
- apc
- home
comments: true
---

After some networking instability in one of my Proxmox cluster nodes, I decided to connect an APC UPS to the node and configure proper UPS monitoring and email notifications.

The UPS model I'm using is:

{% highlight bash %}
root@pve01:~# lsusb | grep American
Bus 003 Device 002: ID 051d:0002 American Power Conversion Uninterruptible Power Supply
{% endhighlight %}

The node where the UPS is connected is `pve01`.

# Install NUT packages

Install the required packages on the Proxmox node:

{% highlight bash %}
root@pve01:~# apt update
root@pve01:~# apt install nut nut-client nut-server mailutils
{% endhighlight %}

# Configure the UPS device

Edit `/etc/nut/ups.conf`:

{% highlight bash %}
root@pve01:~# nano /etc/nut/ups.conf
{% endhighlight %}

Add the UPS definition:

{% highlight ini %}
[apcups]
    driver = usbhid-ups
    port = auto
    pollinterval = 5
    desc = "APC Back-UPS ES 650G2"
{% endhighlight %}

# Configure standalone mode

Edit `/etc/nut/nut.conf`:

{% highlight bash %}
root@pve01:~# nano /etc/nut/nut.conf
{% endhighlight %}

Set:

{% highlight ini %}
MODE=standalone
{% endhighlight %}

# Configure NUT users

Edit `/etc/nut/upsd.users`:

{% highlight bash %}
root@pve01:~# nano /etc/nut/upsd.users
{% endhighlight %}

Add:

{% highlight ini %}
[admin]
    password = strongpassword
    actions = SET
    instcmds = ALL

[monuser]
    password = monitorpassword
    upsmon master
{% endhighlight %}

# Configure UPS monitoring

Edit `/etc/nut/upsmon.conf`:

{% highlight bash %}
root@pve01:~# nano /etc/nut/upsmon.conf
{% endhighlight %}

Use the following configuration:

{% highlight ini %}
MONITOR apcups@localhost 1 monuser monitorpassword master

MINSUPPLIES 1

SHUTDOWNCMD "/usr/bin/sync"

NOTIFYCMD /etc/nut/notify.sh

NOTIFYFLAG ONLINE SYSLOG+EXEC
NOTIFYFLAG ONBATT SYSLOG+EXEC
NOTIFYFLAG LOWBATT SYSLOG+EXEC
NOTIFYFLAG FSD SYSLOG+EXEC
NOTIFYFLAG COMMOK SYSLOG+EXEC
NOTIFYFLAG COMMBAD SYSLOG+EXEC
NOTIFYFLAG SHUTDOWN SYSLOG+EXEC

FINALDELAY 5
HOSTSYNC 15
DEADTIME 30
NOCOMMWARNTIME 300
OFFDURATION 30
POLLFREQ 5
POLLFREQALERT 5
POWERDOWNFLAG "/etc/killpower"
RBWARNTIME 43200
{% endhighlight %}

In my case, I did not want the Proxmox node to power off automatically. Instead, the shutdown command simply flushes pending writes to disk with `sync`.

# Configure email notifications

Create `/etc/nut/notify.sh`:

{% highlight bash %}
root@pve01:~# nano /etc/nut/notify.sh
{% endhighlight %}

Content:

{% highlight bash %}
#!/bin/bash

EMAIL="your@email.com"
HOSTNAME=$(hostname)

case $NOTIFYTYPE in
    ONLINE)
        SUBJECT="[$HOSTNAME] UPS back on utility power"
        BODY="UPS is back ONLINE."
        ;;
    ONBATT)
        SUBJECT="[$HOSTNAME] UPS running on battery"
        BODY="Power failure detected. UPS is ON BATTERY."
        ;;
    LOWBATT)
        SUBJECT="[$HOSTNAME] UPS battery LOW"
        BODY="UPS battery is LOW."
        ;;
    FSD)
        SUBJECT="[$HOSTNAME] UPS forced shutdown"
        BODY="UPS initiated forced shutdown."
        ;;
    COMMBAD)
        SUBJECT="[$HOSTNAME] UPS communication lost"
        BODY="Communication with UPS lost."
        ;;
    COMMOK)
        SUBJECT="[$HOSTNAME] UPS communication restored"
        BODY="Communication with UPS restored."
        ;;
    SHUTDOWN)
        SUBJECT="[$HOSTNAME] UPS shutdown"
        BODY="System shutting down due to UPS event."
        ;;
    *)
        SUBJECT="[$HOSTNAME] UPS event: $NOTIFYTYPE"
        BODY="UPS event detected: $NOTIFYTYPE"
        ;;
esac

echo "$BODY" | mail -s "$SUBJECT" "$EMAIL"
{% endhighlight %}

Make it executable:

{% highlight bash %}
root@pve01:~# chmod +x /etc/nut/notify.sh
{% endhighlight %}

# Configure USB permissions

Create a udev rule:

{% highlight bash %}
root@pve01:~# nano /etc/udev/rules.d/99-nut-ups.rules
{% endhighlight %}

Add:

{% highlight udev %}
SUBSYSTEM=="usb", ATTR{idVendor}=="051d", ATTR{idProduct}=="0002", MODE="0660", GROUP="nut"
{% endhighlight %}

Reload udev rules:

{% highlight bash %}
root@pve01:~# udevadm control --reload-rules
root@pve01:~# udevadm trigger
{% endhighlight %}

Then unplug and reconnect the UPS USB cable.

Verify that permissions are now correct:

{% highlight bash %}
root@pve01:~# ls -l /dev/bus/usb/003/002
crw-rw---- 1 root nut 189, 257 May 17 23:26 /dev/bus/usb/003/002
{% endhighlight %}

# Start NUT services

Start the services cleanly:

{% highlight bash %}
root@pve01:~# systemctl stop nut-monitor
root@pve01:~# systemctl stop nut-server
root@pve01:~# systemctl stop 'nut-driver@apcups.service'

root@pve01:~# pkill -f usbhid-ups || true

root@pve01:~# rm -f /run/nut/usbhid-ups-apcups.pid
root@pve01:~# rm -f /run/nut/usbhid-ups-apcups

root@pve01:~# systemctl start 'nut-driver@apcups.service'
root@pve01:~# systemctl start nut-server
root@pve01:~# systemctl start nut-monitor
{% endhighlight %}

# Verify UPS communication

After everything was configured correctly, the UPS became fully operational.

Verify with:

{% highlight bash %}
root@pve01:~# upsc apcups@localhost
{% endhighlight %}

Example output:

{% highlight bash %}
battery.charge: 100
battery.runtime: 2652
battery.voltage: 13.5
device.model: Back-UPS ES 650G2
driver.name: usbhid-ups
input.voltage: 232.0
ups.load: 12
ups.realpower.nominal: 400
ups.status: OL
{% endhighlight %}

The important field is:

{% highlight bash %}
ups.status: OL
{% endhighlight %}

which means the UPS is online and utility power is present.

# Test notifications

To test notifications manually:

{% highlight bash %}
root@pve01:~# NOTIFYTYPE=ONBATT /etc/nut/notify.sh
{% endhighlight %}

You should receive the notification email immediately.

# Test battery operation

To test the UPS under battery mode, disconnect the UPS from mains power and check the status again:

{% highlight bash %}
root@pve01:~# upsc apcups@localhost
{% endhighlight %}

Example output while on battery:

{% highlight bash %}
battery.charge: 75
battery.runtime: 2571
input.transfer.reason: input voltage out of range
input.voltage: 0.0
ups.load: 10
ups.status: OB DISCHRG
{% endhighlight %}

The important status in this case is:

{% highlight bash %}
ups.status: OB DISCHRG
{% endhighlight %}

Meaning:
- `OB`: On Battery
- `DISCHRG`: Battery is discharging

In this specific test:
- `battery.runtime: 2571`
- means approximately 42 minutes and 51 seconds remaining runtime.

# Wrap up

With this setup:
- Proxmox receives UPS events correctly.
- Email notifications are sent automatically.
- UPS runtime and battery health can be monitored from the shell.
- USB communication survives service restarts correctly.
- The cluster node does not power off automatically, but pending writes are flushed to disk during critical events.

This setup has worked well for monitoring an APC Back-UPS ES 650G2 connected directly to a Proxmox VE node.
