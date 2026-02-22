---
layout: post
status: publish
published: true
title: Debian boot and login screens customization
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2026-02-22 16:50:00 +0100'
tags:
- debian
- trixie
- grub
- lightdm
- chatgpt
comments: true
---
I like to take advantage of the UI customization options that Linux offers.
I'm currently using Debian 13 "Trixie". Sometimes you like the default theme, sometimes you don't.

In any case, I want to tell here two main changes I did to my desktop.

# GRUB default image
In my personal opinion, the default [GRUB](https://en.wikipedia.org/wiki/GNU_GRUB) background image for Debian 13 is a bit sad.

Thus, the steps I took for changing it for something more close to my taste were:

1) GRUB has some limitations.

✅ Supported formats

- PNG (recommended)
- JPG/JPEG
- TGA

PNG works best.

Using a wallpaper from a Lenovo Thinkpad fan page, I created a customized image for this background. Using [GIMP](https://en.wikipedia.org/wiki/GIMP), I created a 1024x768 PNG image, which would suit almost any display at boot time.

You can check your GRUB resolution with:

{% highlight bash %}
# grep GRUB_GFXMODE /etc/default/grub
{% endhighlight %}

If not set, GRUB may default to something like 1024x768.

You can explicitly set resolution:

{% highlight bash %}
GRUB_GFXMODE=1920x1080
GRUB_GFXPAYLOAD_LINUX=keep
{% endhighlight %}

2) Place the image in the right place.

{% highlight bash %}
# cp mybackground.png /boot/grub
{% endhighlight %}

3) Tell GRUB to Use the Image

Edit:

{% highlight bash %}
$ sudo nano /etc/default/grub
{% endhighlight %}

Add or modify this line:

{% highlight bash %}
GRUB_BACKGROUND="/boot/grub/mybackground.png"
{% endhighlight %}

Make sure:

- The path is correct
- Quotes are included
- The file exists

4) Update GRUB

After saving:

{% highlight bash %}
$ sudo update-grub
{% endhighlight %}

This regenerates:

{% highlight bash %}
/boot/grub/grub.cfg
{% endhighlight %}

Reboot to test.

5) (Optional) Enable Graphics Mode if Needed

If the background doesn’t appear, ensure graphics mode is enabled:

In `/etc/default/grub`, verify:

{% highlight bash %}
GRUB_TERMINAL=console
{% endhighlight %}

If it's set to serial or something else, the background won’t show.

If needed, comment it out:

{% highlight bash %}
#GRUB_TERMINAL=console
{% endhighlight %}

Then run:

{% highlight bash %}
sudo update-grub
{% endhighlight %}

# LightDM background image
I'm using [Xfce](https://en.wikipedia.org/wiki/Xfce) for Debian 13 as my default desktop.

[LightDM](https://en.wikipedia.org/wiki/LightDM) is the [display manager] in place, so here is where we have to configure the background image for the login page.

However, I noticed that there is a easier and more straightforward way of changing the desktop theme, altogether.

Debian comes with a list of desktop themes, that you can see by doing

{% highlight bash %}
$ update-alternatives --list desktop-theme
/usr/share/desktop-base/ceratopsian-theme
/usr/share/desktop-base/emerald-theme
/usr/share/desktop-base/futureprototype-theme
/usr/share/desktop-base/homeworld-theme
/usr/share/desktop-base/joy-inksplat-theme
/usr/share/desktop-base/joy-theme
/usr/share/desktop-base/lines-theme
/usr/share/desktop-base/moonlight-theme
/usr/share/desktop-base/softwaves-theme
/usr/share/desktop-base/spacefun-theme
{% endhighlight %}

"Ceratopsian" is the one by default, and that is the one that I don't like.

You can easily go to the folder and see the different images. When you see another that you like, you can just change the default theme, to be system-wide, by doing:

{% highlight bash %}
$ sudo update-alternatives --set desktop-theme /usr/share/desktop-base/futureprototype-theme
update-alternatives: using /usr/share/desktop-base/futureprototype-theme to provide /usr/share/desktop-base/active-theme (desktop-theme) in manual mode
{% endhighlight %}

If you fancy the GRUB image that comes with your theme, just undo the changes done in the GRUB section of this post. You'll get the default image that comes with the selected theme.
