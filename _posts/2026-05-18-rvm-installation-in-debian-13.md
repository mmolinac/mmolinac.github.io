---
layout: post
status: publish
published: true
title: RVM installation in Debian 13
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2026-05-18 10:41:57 +0200'
tags:
- ruby
- rvm
- debian
- jekyll
- openssl
comments: true
---

Recently I needed to continue my Jekyll-based static blog running on a specific Ruby version.
As usual, older Ruby versions tend to become tricky when paired with newer Linux distributions.

In this post I'll go through the exact steps I followed in order to get a working [RVM](https://rvm.io/) installation on Debian 13, including proper OpenSSL support and the dependencies required to build native gems such as `mysql2`.

# Base system requirements

First of all, let's install the packages required to compile Ruby and native extensions:

```bash
sudo apt update
sudo apt install build-essential gcc g++ make \
  libssl-dev zlib1g-dev libreadline-dev \
  libyaml-dev libffi-dev libgdbm-dev \
  libncurses-dev default-libmysqlclient-dev
```

These packages provide the compiler toolchain, OpenSSL headers and the MySQL/MariaDB client development libraries required later by the `mysql2` gem.

# Installing RVM

The recommended way of installing [RVM](https://rvm.io/) is importing the GPG keys and running the installer script:

```bash
gpg --keyserver keyserver.ubuntu.com \
  --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 \
               7D2BAF1CF37B13E2069D6956105BD0E739499BDB

\curl -sSL https://get.rvm.io | bash -s stable
```

After installation, reload the shell environment:

```bash
source ~/.rvm/scripts/rvm
```

And configure RVM to use the system packages instead of custom bundled dependencies:

```bash
rvm autolibs enable
```

# Installing Ruby 3.0.5

My Jekyll project required Ruby `3.0.5`, so I installed that specific version.

Modern Debian releases ship newer OpenSSL versions than the ones expected by Ruby 3.0.x, therefore I used the OpenSSL package provided by RVM itself.

First, install the OpenSSL package managed by RVM:

```bash
rvm pkg install openssl
```

Now install Ruby:

```bash
rvm install ruby-3.0.5 \
  --with-openssl-dir=$HOME/.rvm/usr \
  --disable-binary
```

And make it the default Ruby version:

```bash
rvm use ruby-3.0.5 --default
```

# Verifying OpenSSL support

Once Ruby is installed, we should verify that the OpenSSL extension is correctly available:

```bash
ruby -ropenssl -e 'puts OpenSSL::OPENSSL_VERSION'
```

The output should look similar to:

```text
OpenSSL 1.1.x ...
```

If this command works, HTTPS-based gem installation will also work correctly.

# Installing Bundler

The project lock file required Bundler `2.3.12`, so I installed that exact version:

```bash
gem install bundler:2.3.12
```

We can verify the installation with:

```bash
bundle -v
```

# Installing the project dependencies

At this point we can install all the dependencies required by the Jekyll site:

```bash
bundle _2.3.12_ install
```

This installs all the required gems, including native extensions such as `eventmachine`, `ffi` and `mysql2`.

# Wrap-up

At this point the Jekyll environment is fully operational under Debian 13 using RVM and Ruby 3.0.5.

The most relevant detail is handling the OpenSSL compatibility issue correctly. Older Ruby versions often require building against an older OpenSSL release instead of the one shipped by current Linux distributions.

Once that part is solved, the rest of the Ruby ecosystem works as expected.
