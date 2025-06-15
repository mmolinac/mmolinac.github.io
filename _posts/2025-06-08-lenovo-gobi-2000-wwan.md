---
layout: post
status: publish
published: true
title: Lenovo Thinkpad Gobi 2000 WWAN adapter support under Linux
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2025-06-08 21:07:00 +0200'
tags:
- debian
- lenovo
- wwan
comments: true
---
In my [previous post]({% post_url 2025-06-06-lenovo-x201-fingerprint-debian-12 %}) about one of my Lenovo Thinkpad laptops, I mentioned that it has WWAN capabilities through a [Qualcomm Gobi 2000 adapter](https://www.thinkwiki.org/wiki/Qualcomm_Gobi_2000).

Here I'll put my notes on how to make it work under Linux.

Steps

1. Install the firmware loader for this WWAN modem:
    ```Shell
    $ sudo apt install gobi-loader
    ```

2. Create the folder where we'll host the firmware files:
    ```Shell
    $ sudo mkdir /lib/firmware/gobi
    ```

3. Now you need the original firmware files. Going to Lenovo Support web site, you can download the required files [here](https://support.lenovo.com/us/es/downloads/ds001302). You need to download [7xwc48ww.exe](https://download.lenovo.com/ibmdl/pub/pc/pccbbs/mobiles/7xwc48ww.exe). Install it on a Windows supported system (or at least, extract the files). After that, and following [this table](https://www.thinkwiki.org/wiki/Qualcomm_Gobi_2000#Obtaining_the_Firmware), place the files in the folder created in the previous step:
    ```Shell
    # cp 0/UQCN.mbn UMTS/amss.mbn UMTS/apps.mbn /lib/firmware/gobi/
    ```
    I did it with the Vodafone firmware, but your mileage may vary.

4. Reboot your system.

5. If you did it fine, you'll see the following device now configured:
    ```Shell
    $ lsusb | grep Qualcomm
    Bus 002 Device 006: ID 05c6:9205 Qualcomm, Inc. Gobi 2000
    ```

Additional links to check:
- [Qualcomm Gobi 2000](https://www.thinkwiki.org/wiki/Qualcomm_Gobi_2000). Detailed information coming from ThinkWiki.
- [How to set up Gobi 2000 GPS in Linux](https://github.com/vmikhailenko/gobictl)
- [X201 Qualcomm und Win 10](https://thinkpad-forum.de/threads/x201-qualcomm-und-win-10.189978/). It's in German. You'll find there how to get support for this device under Windows 10. I can tell you where to find the file `win8beta_7xwc45ww.zip` that is also valid to use it under Windows 11.