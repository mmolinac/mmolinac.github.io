---
layout: post
status: publish
published: true
title: Booting Linux through OCLP
author: Manuel Molina
about_author: "/us/manuel/readme.html"
layout: post
status: publish
published: true
date: '2021-12-19 15:51:27 +0100'
author_email: mmc@pocosmhz.org
tags:
- macOS
comments: []
---
There are several ways of booting Linux in a system where you've installed [OpenCore Legacy Patcher](https://dortania.github.io/OpenCore-Legacy-Patcher/).

The one that best suited my needs was [this one](https://dortania.github.io/OpenCore-Multiboot/oc/linux.html#method-a-openlinuxboot), and allowed me to end up with the following boot menu:

![OCLP boot menu](/content/images/2021-12-19-booting-linux-through-oclp/oclp_boot_menu-1024x514.jpg)

Please be aware that you must provide the file `ext4_x64.efi` that is mentioned there, and put it inside folder `/EFI/OC/Drivers` .

That driver can be found on any distribution of [rEFInd](http://www.rodsbooks.com/refind/index.html).

At the same time, you must modify the file `EFI/OC/config.plist` and add the two following driver sections:

{% highlight XML %}
            <dict>
                <key>Arguments</key>
                <string></string>
                <key>Comment</key>
                <string></string>
                <key>Enabled</key>
                <true/>
                <key>Path</key>
                <string>OpenLinuxBoot.efi</string>
            </dict>
            <dict>
                <key>Arguments</key>
                <string></string>
                <key>Comment</key>
                <string></string>
                <key>Enabled</key>
                <true/>
                <key>Path</key>
                <string>ext4_x64.efi</string>
            </dict>
{% endhighlight %}

Be aware, as well, that if you reinstall OpenCore Legacy Patcher (because you wanted to update its version or other reasons), you must repeat the process.
