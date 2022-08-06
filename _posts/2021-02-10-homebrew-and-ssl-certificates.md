---
layout: post
title: Homebrew and SSL certificates
author: Manuel Molina
about_author: "/us/manuel/readme.html"
layout: post
status: publish
published: true
author_email: mmc@pocosmhz.org
tags:
- bigdata
comments: []
---
There's an issue with some packages in Homebrew regarding SSL certificates. Once in a while I need to remember which env variable do I have to set in order to install its upgrades with Homebrew. [Libreoffice](https://www.libreoffice.org/) is one of them.

[This](https://stackoverflow.com/a/57655105) is the answer in Stack Overflow:

{% highlight bash %}
echo insecure >> ~/.curlrc
HOMEBREW_CURLRC=1
export HOMEBREW_CURLRC
brew install --cask libreoffice
{% endhighlight %}

This way we'll force Homebrew to read our preferences whenever it runs `curl` .
