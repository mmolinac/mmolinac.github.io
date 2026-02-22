---
layout: post
status: publish
published: true
title: NordPass login issue under Debian
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2026-02-22 18:06:00 +0100'
tags:
- debian
- trixie
- nordpass
- chatgpt
comments: true
---
I've been using [NordPass](https://nordpass.com) as my password manager of choice for some years now.

It works like a charm under Mac and Ubuntu Linux. However, a different issue when it came to Debian as my new desktop of choice.

Thanks to ChatGPT I've been able to fix an issue that has been annoying me for months.

Under Linux, no matter what, NordPass comes as a [Snap application](https://en.wikipedia.org/wiki/Snap_(software)). Once you have installed it and start the app, it will take you to the default web browser so you can authenticate yourself in their website.

Once you're authenticated, you're offered a link of the form `nordpass://vault?action=login&status=done&verify_token=dfc088f3ab8d5be4b554d5d994dffc4cb5c0fb2e676bcf427b6322c4c3bb2f62` that you can open, so it takes you back to the desktop app.

Well, that link does open under Ubuntu, but not under Debian. The reason? This is a custom protocol handler, and my suspicion is that Firefox ESR is not allowed to open external protocol handlers *by default*.

The steps you have to follow to make that happen are:

1) Check if Debian knows the protocol

Run:

{% highlight bash %}
xdg-mime query default x-scheme-handler/nordpass
{% endhighlight %}

If nothing is returned → that’s the problem.
NordPass hasn’t registered itself properly.

2) — Manually register NordPass as the handler

First find the Snap desktop file:

{% highlight bash %}
ls /var/lib/snapd/desktop/applications | grep -i nordpass
{% endhighlight %}

You should see something like:

```
snap.nordpass.nordpass.desktop
```

Now register it:

{% highlight bash %}
xdg-mime default snap.nordpass.nordpass.desktop x-scheme-handler/nordpass
{% endhighlight %}

Then update desktop database:

{% highlight bash %}
update-desktop-database ~/.local/share/applications
{% endhighlight %}

Now test:

{% highlight bash %}
xdg-open "nordpass://test"
{% endhighlight %}

If NordPass opens → fixed.

3) If it still fails (Snap sandbox issue)

Snap apps sometimes cannot receive custom URL callbacks because of missing portal integration.

Check if the snap has desktop integration:

{% highlight bash %}
snap connections nordpass
{% endhighlight %}

Look for:

desktop
desktop-legacy
xdg-desktop-portal

If not connected:

{% highlight bash %}
sudo snap connect nordpass:desktop
sudo snap connect nordpass:desktop-legacy
{% endhighlight %}

4) Firefox setting check

In Firefox:

- Go to: `about:config`
- Search: `network.protocol-handler.expose.nordpass`
- If it exists and is set to true, change it to false.
- If it doesn’t exist, create:
  - Type: Boolean
  - Name: network.protocol-handler.expose.nordpass
  - Value: false
- Restart Firefox.

This tells Firefox it is allowed to open external handlers.

🔎 Why this happens

Snap applications:

- Run sandboxed
- Sometimes fail to register URL schemes properly

Debian 13 is stricter with portals and desktop integration

This is not a NordPass login problem — it's a custom protocol handler registration issue.