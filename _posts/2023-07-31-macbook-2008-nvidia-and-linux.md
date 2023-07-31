---
layout: post
status: publish
published: true
title: MacBook 2008, Nvidia and Linux
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2023-07-31 16:31:45 +0100'
tags:
- macOS
- Linux
comments: true
---
After my [previous post]({% post_url 2021-12-19-keeping-my-macbook-2008-alive-and-kicking %}) on the matter of keeping my old MacBook usable, there have been some nice developments.

I replaced the SSD hard drive I had with a bigger one. Also, I wanted to use one which is also compatible with the picky internal SATA controller (see [here](https://forums.macrumors.com/threads/ssds-sata-and-negotiated-link-speeds.2047851/) about the issue).
My choice was a Crucial MX500 SSD drive. It's able to negotiate the right speed with the SATA II controller:

{% highlight bash %}
# dmesg | grep -i --color ahci | grep Gbps
[    4.201832] ahci 0000:00:0b.0: AHCI 0001.0200 32 slots 6 ports 3 Gbps 0x3 impl SATA mode

# smartctl -a /dev/sda | egrep "(Model|SATA Version)"
Model Family:     Crucial/Micron Client SSDs
Device Model:     CT500MX500SSD1
SATA Version is:  SATA 3.3, 6.0 Gb/s (current: 3.0 Gb/s)
{% endhighlight %}

However, coming to a more *visible* issue on Linux, let me tell what my graphics driver journey have been so far.

In the past, there were maintained proprietary Linux drivers provided by Nvidia. For this particular laptop and card, NVidia GeForce 9400M, you can find them [here](https://www.nvidia.com/download/driverResults.aspx/156163/en-us/).

However, as you can notice by complains like [this](https://askubuntu.com/questions/1453326/22-04-1-and-nvidia-legacy-driver-vs-nouveau) or [this one](https://bbs.archlinux.org/viewtopic.php?id=279064), you won't go too far if you want your system updated. That happened to me and it is the main reason I had to stop using them.

The other path (and solution for me!) is to move to the [Nouveau](https://nouveau.freedesktop.org/) drivers. However, once you do that, you can find messages like this:

{% highlight bash %}
# dmesg | grep -B 3 -i "init failed"
Jul 27 22:26:25 pacharan kernel: [   77.339154] nouveau 0000:02:00.0: Direct firmware load for nouveau/nvac_fuc084 failed with error -2
Jul 27 22:26:25 pacharan kernel: [   77.339200] nouveau 0000:02:00.0: Direct firmware load for nouveau/nvac_fuc084d failed with error -2
Jul 27 22:26:25 pacharan kernel: [   77.339204] nouveau 0000:02:00.0: msvld: unable to load firmware data
Jul 27 22:26:25 pacharan kernel: [   77.339210] nouveau 0000:02:00.0: msvld: init failed, -19
{% endhighlight %}

Other users had been reporting that, like in [this post](https://unix.stackexchange.com/questions/677112/errors-from-nouveau-display-driver-on-debian-firmware-failed-to-load-nouveau).
At first I was a bit lost. I recently realized that this error was related to *video acceleration*. I found [this article](https://nouveau.freedesktop.org/VideoAcceleration.html) regarding the matter at hand.

I followed the directions and did just that:

{% highlight bash %}
# mkdir /tmp/nouveau
# wget https://raw.github.com/envytools/firmware/master/extract_firmware.py
# wget https://us.download.nvidia.com/XFree86/Linux-x86_64/340.108/NVIDIA-Linux-x86_64-340.108.run
# sh NVIDIA-Linux-x86-325.15.run --extract-only
# python3 extract_firmware.py
# mkdir -p /lib/firmware/nouveau
# cp -d nv* vuc-* /lib/firmware/nouveau/
{% endhighlight %}

After rebooting, I get this:

{% highlight bash %}
# dmesg | grep nouvea| grep -i initi
[    5.263711] [drm] Initialized nouveau 1.3.1 20120801 for 0000:02:00.0 on minor 0
{% endhighlight %}

... And as they said in the page, *You should be ready to use video decoding.*