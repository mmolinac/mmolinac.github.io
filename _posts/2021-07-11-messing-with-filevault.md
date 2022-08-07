---
layout: post
status: publish
published: true
title: Messing with FileVault
author: Manuel Molina
about_author: "/us/manuel/readme.html"
layout: post
status: publish
published: true
author_email: mmc@pocosmhz.org
tags:
- macOS
comments: []
---
I know I have your attention. Well, I've been doing just that, because I'm using macOS Catalina in my old (but working!) [MacBook Aluminum Unibody](https://en.wikipedia.org/wiki/MacBook_(2006%E2%80%932012)#2nd_generation:_Aluminum_Unibody)

This is not a _supported configuration_, so I'm not on the safe side here...

I installed that unsupported version thanks to the [macOS Catalina Patcher](http://dosdude1.com/catalina/) tool by [dosdude1](http://twitter.com/dosdude1). It just works fine, beside having some trouble bringing and installing latest OTA updates. For that task, I've been \{using\|messing with\} [CatalinaOTAswufix](https://github.com/jacklukem/CatalinaOTAswufix), from the awesome [jackluke](https://forums.macrumors.com/members/1133911/).

As much as I like both tools, it looks like you can't get everything working at the same time in certain circumstances. That's life.

I was having a working configuration until some minutes ago: macOS Catalina 19H15 working in a MacBook from 2008. I wanted to enable FileVault as I had with Mojave. After encryption, rebooted, and the computer hung up. How am I going to fix this? I can't login to the computer anymore.

Google came to help me. I found [this great article](https://derflounder.wordpress.com/2019/01/15/unlock-or-decrypt-your-filevault-encrypted-boot-drive-from-the-command-line-on-macos-mojave/) about how to do FileVault tasks command-line in Mojave, that also applies here. Then:

- I booted from my patched Catalina installation USB media in Rescue Mode.
- Mounted the encrypted volume through Disk Utility.
- Open a Terminal and followed the directions to decrypt the vaulted volumes.

One funny thing: Once you do that to the first of the two encrypted volumes, there's some sort of link between them (system and data volumes), and they start decrypting at the same time.

Once the decryption process finished, I reinstalled the working patched version 19H15 of macOS Catalina and rebooted the computer. And off you go!
