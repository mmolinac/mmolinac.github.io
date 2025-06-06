---
layout: post
status: publish
published: true
title: Lenovo Thinkpad X201 fingerprint sensor and Debian 12
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2025-06-06 16:45:00 +0200'
tags:
- debian
- lenovo
- fingerprint
comments: true
---
On December last year I manage to build a special Lenovo Thinkpad X201 with some very specific treats:
- 3G embedded support (Qualcomm, Inc. Qualcomm Gobi 2000)
- Lenovo Integrated Webcam
- Upek Biometric Touchchip/Touchstrip Fingerprint Sensor
- Gemalto (was Gemplus) Compact Smart Card Reader Writer

It was cheap, fun to configure and easy to carry whenever I needed to go.

In this post I'm going to describe how I configured the fingerprint sensor.

For Microsoft Windows 11 you don't have to do anything special, as the device is automatically configured. You just go to your user preferences and add fingerprint logging and follow the procedure to enroll your fingerprint(s).


For Debian 12, I'll describe the quick procedure below.
But before that, please let me cite the original blog entry of the device driver author [here](http://www.reactivated.net/weblog/archives/2008/07/upek-touchstrip-sensor-only-147e2016-on-linux/).

Now with the procedure:

1. Install some Debian packages:
    ```Shell
    $ sudo apt install fprintd libpam-fprintd
    ``` 

2. Configure PAM (Pluggable Authentication Modules):
    ```Shell
    $ sudo pam-auth-update
    ```
    In the PAM configuration menu, select "Fingerprint authentication" and click OK.
    ```
    [*] Fingerprint authentication
    [*] Unix authentication
    [*] Register user sessions in the systemd control group hierarchy
    [ ] Create home directory on login
    [*] GNOME Keyring Daemon - Login keyring management

    <Ok>    <Cancel>
    ```

3. Enroll Your Fingerprint.
    After restarting your system, you should be able to enroll your fingerprint. Use the `fprint-enroll` tool or a graphical interface (if available) to enroll your fingerprint.
    ```Shell
    $ fprintd-enroll 
    Using device /net/reactivated/Fprint/Device/0
    Enrolling right-index-finger finger.
    Enroll result: enroll-stage-passed
    Enroll result: enroll-stage-passed
    Enroll result: enroll-stage-passed
    Enroll result: enroll-stage-passed
    Enroll result: enroll-stage-passed
    Enroll result: enroll-completed
    ```
    You will need several passes for the fingerprint to be completely enrolled. Repeat until you see the last message.

4. Now you can log in to your system using your login manager, like [LightDM](https://github.com/canonical/lightdm) in my case.
    Also, you can use your fingerprint to authenticate yourself in other situations:
    ```Shell
    $ sudo su -
    Swipe your right index finger across the fingerprint reader
    # 
    ```

5. Optionally, you can enroll other fingerprints or manage them through command-line utilities:
    ```Shell
    asdf

    ```



    $ fprintd-list manuelmc
    found 1 devices
    Device at /net/reactivated/Fprint/Device/0
    Using device /net/reactivated/Fprint/Device/0
    Fingerprints for user manuelmc on Upek TouchChip Fingerprint Coprocessor (swipe):
    - #0: right-index-finger
    ```
