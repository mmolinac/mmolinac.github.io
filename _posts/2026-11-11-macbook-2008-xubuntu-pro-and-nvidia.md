---
layout: post
status: draft
published: true
title: "MacBook 2008 revisited: Xubuntu 24.04, Ubuntu Pro and NVIDIA"
author: Manuel Molina
about_author: "/us/manuel/readme.html"
date: '2026-11-11 12:00:00 +0100'
tags:
- Linux
- Ubuntu
- NVIDIA
comments: true
---

In my [2023 post]({% post_url 2023-07-31-macbook-2008-nvidia-and-linux %}), I had already put a Crucial MX500 SSD in my late-2008 MacBook (MacBook5,1). It negotiated the full 3 Gb/s of the laptop's SATA link. The graphics were another story. I had moved to Nouveau after the proprietary NVIDIA driver stopped keeping up with newer kernels, and I had to track down missing firmware for video decoding.

By October 2026 the MacBook was running Xubuntu 24.04.5 LTS with Nouveau. I wanted to keep that installation, get security updates for the Ubuntu packages over the long term, and see if I could get the proprietary driver working again on the GeForce 9400M.

## Cleaning up APT

This installation had picked up a few third-party repositories over the years. The Ubuntu sources were already on Noble, but the Google Cloud configuration still referenced Bionic and Xenial, VirtualBox pointed to Eoan, and ownCloud used a Debian 13 repository. Spotify was listed twice, and some repositories had signing-key errors. The Ookla repository returned a 404 for Noble.

I backed up `/etc/apt`, switched Google Cloud to its `cloud-sdk` repository and pointed ownCloud and VirtualBox at their Ubuntu 24.04 repositories. I used HTTPS and per-repository `Signed-By` entries for the sources I repaired, kept one Spotify entry and disabled the Ookla source. The installed Speedtest command stayed in place. Changing the VirtualBox source also left the existing VirtualBox 6.1 installation in place. After a later Spotify update recreated the duplicate source, I disabled it again.

{% highlight bash %}
sudo apt-get update --error-on=any
{% endhighlight %}

That gave me a clean APT refresh before I attached the machine to Ubuntu Pro.

## Enrolling in Ubuntu Pro

I signed up for [Ubuntu Pro](https://ubuntu.com/pro) using the free personal subscription and got an attachment token from the dashboard. On the MacBook I ran the following, using my own token in place of `YOUR_TOKEN`.

{% highlight bash %}
sudo pro attach YOUR_TOKEN
pro status
{% endhighlight %}

A power outage cut my connection while the attach command was enabling ESM Infra. When the machine came back, `pro status` showed `esm-apps`, `esm-infra` and `livepatch` all enabled. A fresh APT update reached the ESM repositories, and the upgrade simulation listed 47 ESM security updates with no removals.

[Xubuntu 24.04's flavor support runs through April 2027](https://xubuntu.org/release/24-04/). [Ubuntu Pro extends security maintenance for eligible Ubuntu 24.04 packages through May 2034](https://ubuntu.com/about/release-cycle). My NVIDIA driver comes from a community PPA, so its compatibility with future kernels depends on that project's patches.

## Bringing back NVIDIA 340

The GeForce 9400M in this machine identifies as `10de:0863`. The driver I used was NVIDIA **340.108**, originally released on [23 December 2019](https://www.nvidia.com/download/driverResults.aspx/156163/en-us/). The [community packaging for Ubuntu 24.04](https://github.com/kda2210/nvidia-340-ubuntu-24.04) builds that driver against newer kernels using DKMS. My working combination was package version `340.108-26` and kernel `6.8.0-147-generic`.

{% highlight bash %}
sudo add-apt-repository ppa:kda2210/nvidia340
sudo apt update
sudo apt install nvidia340

dkms status
modinfo -k "$(uname -r)" -F version nvidia-legacy-340xx
{% endhighlight %}

The installer warned that Nouveau was loaded and that I needed to reboot. Before rebooting I checked that DKMS had installed the module for my running kernel and that `modinfo` returned `340.108`. The name to use with `modinfo` on this package was `nvidia-legacy-340xx`. Once loaded, the module appeared as `nvidia`.

After rebooting, I checked the driver and ran `glxinfo` from a terminal in the local Xfce session. The `glxinfo` command is provided by `mesa-utils`.

{% highlight bash %}
lspci -nnk | grep -A3 -Ei 'VGA|3D|Display'
lsmod | grep -E 'nvidia|nouveau'
glxinfo -B
{% endhighlight %}

The PCI listing showed `Kernel driver in use: nvidia`, and `lsmod` showed `nvidia` loaded. The relevant OpenGL lines were:

{% highlight text %}
direct rendering: Yes
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: GeForce 9400M/integrated/SSE2
OpenGL core profile version string: 3.3.0 NVIDIA 340.108
OpenGL ES profile version string: OpenGL ES 2.0 NVIDIA 340.108 340.108
{% endhighlight %}

## Getting the brightness keys back

The desktop came up, but the brightness keys did nothing. The kernel command line already included `acpi_backlight=vendor`. The `/sys/class/backlight/` directory was empty, and loading `apple_bl` didn't add a device. Xfce's power manager was already running.

The NVIDIA module exposed an `NVreg_RegistryDwords` parameter, and `/proc/driver/nvidia/params` showed an empty value. I enabled NVIDIA's brightness control through that parameter and rebuilt the initramfs for the current kernel.

{% highlight bash %}
printf '%s\n' \
  'options nvidia-legacy-340xx NVreg_RegistryDwords=EnableBrightnessControl=1' \
  | sudo tee /etc/modprobe.d/nvidia-340-backlight.conf

sudo update-initramfs -u -k "$(uname -r)"
sudo reboot
{% endhighlight %}

After logging in again, pressing the keys brought up the Xfce slider and visibly dimmed or brightened the panel. The module reported the setting below, even though `/sys/class/backlight/` remained empty.

{% highlight bash %}
grep -i RegistryDwords /proc/driver/nvidia/params
{% endhighlight %}

{% highlight text %}
RegistryDwords: "EnableBrightnessControl=1"
{% endhighlight %}

## How it runs now

YouTube played smoothly at the higher quality I tried. That was a playback test, and I didn't measure whether the GPU was decoding the video. PDFCraft worked without a rendering warning, while PhotoCraft fell back to CPU rendering. The GeForce 9400M has working OpenGL on this setup, but it has no hardware Vulkan support.

The MacBook now has a responsive Xfce desktop, working brightness keys and Ubuntu Pro enabled. The upgrade prompt offered Ubuntu 26.04.1 LTS, but I'm staying on 24.04 for now. I have a working driver on this kernel, and I'll revisit the upgrade once I can verify a build for the newer release.
