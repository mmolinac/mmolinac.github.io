---
layout: post
status: publish
published: true
title: Proxmox major version upgrade
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2025-12-23 19:20:00 +0200'
tags:
- hypervisor
- home
- ha
- budget
- proxmox
comments: true
---
Some months after [starting using Proxmox]({% post_url 2025-04-13-proxmox-home-cluster-i %}) 8, it's time to upgrade to [Proxmox 9](https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-0).

We'll be using the [in place upgrade](https://pve.proxmox.com/wiki/Upgrade_from_8_to_9#In-place_upgrade) method. I'm going to take you through the steps I took to upgrade my home cluster.

# Upgrade all nodes to latest minor version
From current major version 8, be sure that all nodes are already on this minor version:

{% highlight bash %}
root@pve01:~# pveversion
pve-manager/8.4.14/b502d23c55afcba1 (running kernel: 6.8.12-17-pve)
{% endhighlight %}

# Check prerequisites
Double check that you have a correct Ceph status (if shared storage is in use) and a healthy PVE cluster, as detailed in the Prerequisites section of the upgrade documentation.

# pve8to9 checklist script
Run the script on all nodes prior to start any upgrade. Be sure that you have no errors or hard notices from it.

I had the following message, and similar ones have to be fixed before starting the upgrade:

{% highlight bash %}
root@pve01:~# pve8to9 |& grep -i FAIL
FAIL: systemd-boot meta-package installed. This will cause problems on upgrades of other boot-related packages. Remove 'systemd-boot' See https://pve.proxmox.com/wiki/Upgrade_from_8_to_9#sd-boot-warning for more information.
FAILURES: 1
{% endhighlight %}

In my case, it was safe to do this on all nodes:

{% highlight bash %}
root@pve01:~# apt-get remove systemd-boot
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  proxmox-kernel-6.8.12-13-pve-signed proxmox-kernel-6.8.12-14-pve-signed
Use 'apt autoremove' to remove them.
The following packages will be REMOVED:
  systemd-boot
0 upgraded, 0 newly installed, 1 to remove and 0 not upgraded.
After this operation, 250 kB disk space will be freed.
Do you want to continue? [Y/n] Y
(Reading database ... 80236 files and directories currently installed.)
Removing systemd-boot (252.39-1~deb12u1) ...
Processing triggers for man-db (2.11.2-2) ...
{% endhighlight %}

# Put a node in maintenance mode
We'll start performing the upgrade to one node, and we'll repeat these steps on every node, one at a time.
We'll always wait for the cluster to be 100% available before moving on to the next node upgrade.

To put a node in maintenance mode, go to the shell and do:

{% highlight bash %}
root@pve01:~# ha-manager crm-command node-maintenance enable pve01
{% endhighlight %}

After VMs and CMs have been migrated, we can see there are none in the node we're about to upgrade:

{% highlight bash %}
root@pve01:~# ha-manager status
quorum OK
master pve03 (active, Tue Dec 23 19:56:08 2025)
lrm pve01 (maintenance mode, Tue Dec 23 19:56:09 2025)
lrm pve02 (active, Tue Dec 23 19:56:07 2025)
lrm pve03 (active, Tue Dec 23 19:56:04 2025)
service vm:100 (pve03, started)
service vm:101 (pve02, started)
service vm:102 (pve02, started)
service vm:104 (pve03, started)
service vm:105 (pve02, started)
{% endhighlight %}

# Perform software upgrade
This step is split in several sub-steps.

## Update Debian Base Repositories to Trixie
Run the following to update the base OS repositories:

{% highlight bash %}
root@pve01:~# sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
root@pve01:~# sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/pve-enterprise.list
{% endhighlight %}

## Add the Proxmox VE 9 Package Repository
In my case, I'm using the no-subscription repository, so I do this:

{% highlight bash %}
root@pve01:~# cat > /etc/apt/sources.list.d/proxmox.sources << EOF
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
{% endhighlight %}

In any case, double check that you're not leaving behind traces of Proxmox 8 repositories and that the new ones work, through `apt update`. There could be some traces in file `/etc/apt/sources.list`.

## Update the Ceph Package Repository
In case you're using Ceph shared storage, you must also update the Ceph package repository.

In my case, I'm using the no-subscription repository, so I do this:

{% highlight bash %}
root@pve01:~# cat > /etc/apt/sources.list.d/ceph.sources << EOF
Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
{% endhighlight %}

Also, I made sure no traces of the previous Ceph repository are there, by doing:

{% highlight bash %}
root@pve01:~# rm /etc/apt/sources.list.d/ceph.list 
{% endhighlight %}

## Refresh package index
Run the command and make sure you don't have any errors:

{% highlight bash %}
root@pve01:~# apt update
Hit:1 http://ftp.es.debian.org/debian trixie InRelease
Hit:2 http://security.debian.org trixie-security InRelease                 
Hit:3 http://ftp.es.debian.org/debian trixie-updates InRelease             
Hit:4 http://security.debian.org/debian-security trixie-security InRelease 
Hit:5 http://download.proxmox.com/debian/ceph-squid trixie InRelease
Hit:6 http://download.proxmox.com/debian/pve trixie InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
639 packages can be upgraded. Run 'apt list --upgradable' to see them.
{% endhighlight %}

## Upgrade the system to Debian Trixie and Proxmox VE 9.0
The following command could take quite some time, so be aware:

{% highlight bash %}
root@pve01:~# apt dist-upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages were automatically installed and are no longer required:
{% endhighlight %}
[...]
{% highlight bash %}
639 upgraded, 155 newly installed, 61 to remove and 0 not upgraded.
Need to get 886 MB of archives.
After this operation, 1,729 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
{% endhighlight %}
Say yes and go ahead with the upgrade.

There will be some questions about configuration changes to be overridden or not by the incoming packages. Answer according to your preferences.

## Check Result & Reboot Into Updated Kernel
If the previous command exited without error, you can re-check with `pve8to9` and confirm that everything is in place for a reboot.

{% highlight bash %}
root@pve01:~# sync ; init 6 ; exit
{% endhighlight %}

## Post-upgrade actions
When you log in to the upgraded cluster node, please check:
- Cluster status is OK.
- Proxmox VE 9 deprecates HA groups in favor of HA rules. If you are using HA and HA groups, HA groups will be automatically migrated to HA rules once all cluster nodes have been upgraded to Proxmox VE 9.

Now you can disable maintenance mode by doing:
{% highlight bash %}
root@pve01:~# ha-manager crm-command node-maintenance disable pve01
{% endhighlight %}

Optionally, you can take the change to normalize any package source archive that is still with the old format, by doing:
{% highlight bash %}
root@pve01:~# apt modernize-sources
The following files need modernizing:
  - /etc/apt/sources.list
  - /etc/apt/sources.list.d/pve-enterprise.list

Modernizing will replace .list files with the new .sources format,
add Signed-By values where they can be determined automatically,
and save the old files into .list.bak files.

This command supports the 'signed-by' and 'trusted' options. If you
have specified other options inside [] brackets, please transfer them
manually to the output files; see sources.list(5) for a mapping.

For a simulation, respond N in the following prompt.
Rewrite 2 sources? [Y/n] Y
Modernizing /etc/apt/sources.list...
- Writing /etc/apt/sources.list.d/debian.sources

Modernizing /etc/apt/sources.list.d/pve-enterprise.list...
{% endhighlight %}

**Be aware that any commented entries would be uncommented, so double check with `apt update` after that and act accordingly.**

# Wrap up
After you've done the upgrade procedure on all nodes, do a few final checks.

Check cluster health:
{% highlight bash %}
root@pve03:~# pvecm status
Cluster information
-------------------
Name:             myclust
Config Version:   3
Transport:        knet
Secure auth:      on

Quorum information
------------------
Date:             Wed Dec 24 02:20:02 2025
Quorum provider:  corosync_votequorum
Nodes:            3
Node ID:          0x00000003
Ring ID:          1.20c
Quorate:          Yes

Votequorum information
----------------------
Expected votes:   3
Highest expected: 3
Total votes:      3
Quorum:           2  
Flags:            Quorate 

Membership information
----------------------
    Nodeid      Votes Name
0x00000001          1 192.168.18.131
0x00000002          1 192.168.18.132
0x00000003          1 192.168.18.133 (local)
{% endhighlight %}

Check HA status:
{% highlight bash %}
root@pve03:~# ha-manager status
quorum OK
master pve02 (active, Wed Dec 24 02:22:06 2025)
lrm pve01 (active, Wed Dec 24 02:22:03 2025)
lrm pve02 (active, Wed Dec 24 02:22:04 2025)
lrm pve03 (active, Wed Dec 24 02:21:59 2025)
service vm:100 (pve03, started)
service vm:101 (pve01, started)
service vm:102 (pve02, started)
service vm:104 (pve03, started)
service vm:105 (pve01, started)
{% endhighlight %}

Check Ceph (shared storage) status:
{% highlight bash %}
root@pve03:~# pveceph status
  cluster:
    id:     f35872da-c5a3-4599-af36-b99c2b64c0f3
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum pve01,pve02,pve03 (age 16m)
    mgr: pve03(active, since 16m), standbys: pve01, pve02
    osd: 3 osds: 3 up (since 15m), 3 in (since 4M)
 
  data:
    pools:   3 pools, 65 pgs
    objects: 28.56k objects, 106 GiB
    usage:   307 GiB used, 1.8 TiB / 2.1 TiB avail
    pgs:     65 active+clean
 
  io:
    client:   0 B/s rd, 450 KiB/s wr, 0 op/s rd, 91 op/s wr
{% endhighlight %}

With that `HEALTH_OK`, we're done.