---
layout: post
status: publish
published: true
title: Old Ruby version installation with RVM
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2025-06-06 16:07:00 +0200'
tags:
- ruby
- rvm
- openssl
comments: true
---
This will be a quick post on how to quickly install an old Ruby version on your Linux or Mac computer, with the help of [RVM](https://rvm.io/), the Ruby Version Manager:

0. First and foremost, if you haven't already, let's install RVM:
    For the current user (no system-wide installation), you have to:
    ```Shell
    \curl -sSL https://get.rvm.io | bash -s stable --ruby
    ```

1. Once you have RVM installed, we'll request the installation of an old Ruby version, with its quirks and things:
    ```Shell
    $ rvm pkg install openssl
    ```

2. With that dependency installed, we're now able to install the selected Ruby version with OpenSSL support:
    ```Shell
    $ rvm install ruby-3.0.5 --with-openssl-dir=$HOME/.rvm/usr
    ```
    It will take a while.

3. With the version installed, let's install [Bundler](https://bundler.io/):
    ```Shell
    $ rvm all do gem install bundler
    ```

4. Let's use the newly installed Ruby version:
    ```Shell
    $ rvm use ruby-3.0.5

    RVM is not a function, selecting rubies with 'rvm use ...' will not work.

    You need to change your terminal emulator preferences to allow login shell.
    Sometimes it is required to use `/bin/bash --login` as the command.
    Please visit https://rvm.io/integration/gnome-terminal/ for an example.
    ```
    Oops! It looks we need to allow login shell for the macros to work:
    ```Shell
    $ bash --login
    $ rvm use ruby-3.0.5
    Using /home/manuelmc/.rvm/gems/ruby-3.0.5
    ```
