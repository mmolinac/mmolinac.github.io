---
layout: post
status: publish
published: true
title: Keeping my MacBook 2008 alive and kicking
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2021-12-19 22:28:59 +0100'
tags:
- macOS
- brew
comments: true
---
![MacBook 2008 unibody](/content/images/2021-12-19-keeping-my-macbook-2008-alive-and-kicking/macbook2008alu.jpg)

In order to continue being able to use and work with my old but working [MacBook 2008 Unibody](https://en.wikipedia.org/wiki/MacBook_(2006%E2%80%932012)#1st_generation:_Aluminum_Unibody), I have two main hurdles:

- To keep up with the OS upgrades. Apple no longer support this laptop with their new OS releases, so I started solving this with [dosdude1](https://twitter.com/dosdude1)'s [Mojave](http://dosdude1.com/mojave/) and [Catalina](http://dosdude1.com/catalina/) patchers. Now I do it by using OpenCore Legacy Patcher to install Big Sur and, in the future, Monterey.
- To have an up to date stack of software and utilities. For the most part I rely on [brew](https://brew.sh/) to do that.

However, **brew** software is continuously optimized for the latest macOS version. This has lately implied generating [bottled packages](https://docs.brew.sh/Bottles) with the optimizations available for the oldest macOS supported by Apple.

And then one day, while trying to use a bottled version of Python, I bumped into this error:

{% highlight bash %}
$ /usr/local/Cellar/python\@3.8/3.8.9/bin/python3 -V
Illegal instruction: 4
$  
{% endhighlight %}

After some research, I found reports like [this one](https://github.com/conda/conda/issues/9678) or [this one](https://github.com/Homebrew/legacy-homebrew/issues/19567). However, the definitive answer came from [this](https://github.com/d12frosted/homebrew-emacs-plus/issues/368#issuecomment-928350664) GitHub comment from user [@koenige50](https://github.com/koenige50).

I followed the directions in order to make my installation of brew started allowing me to compile software optimized for the [Core 2 Duo](https://en.wikipedia.org/wiki/Intel_Core_2) architecture of my MacBook. The content of the relevant file is now:

{% highlight Ruby %}
$ cat /usr/local/Homebrew/Library/Homebrew/extend/os/mac/hardware.rb
# typed: strict
# frozen_string_literal: true

module Hardware
  extend T::Sig
  sig { params(version: T.nilable(Version)).returns(Symbol) }
  def self.oldest_cpu(version = nil)
    version = if version
      MacOS::Version.new(version.to_s)
    else
      MacOS.version
    end
    if CPU.arch == :arm64
      :arm_vortex_tempest
    # TODO: this cannot be re-enabled until either Rosetta 2 supports AVX
    # instructions in bottles or Homebrew refuses to run under Rosetta 2 (when
    # ARM support is sufficiently complete):
    #   https://github.com/Homebrew/homebrew-core/issues/67713
    #
    # elsif version >= :big_sur
    #   :ivybridge
    elsif version >= :mojave
      :core2
    else
      generic_oldest_cpu
    end
  end
end
{% endhighlight %}

As you can see, I've replaced `:nehalem` with `:core2` in line 24.

Now, for every conflictive formula that I found, I've only have to do this:

{% highlight bash %}
$ brew reinstall --build-from-source node@12
==> Downloading https://nodejs.org/dist/v12.22.8/node-v12.22.8.tar.xz
######################################################################## 100.0%
==> Reinstalling node@12 
==> python3 configure.py --prefix=/usr/local/Cellar/node@12/12.22.8 --with-intl=system-icu --shared-libuv --shared-n
==> make install
==> Caveats
node@12 is keg-only, which means it was not symlinked into /usr/local,
because this is an alternate version of another formula.

If you need to have node@12 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/node@12/bin:$PATH"' >> /Users/manuelmc/.bash_profile

For compilers to find node@12 you may need to set:
  export LDFLAGS="-L/usr/local/opt/node@12/lib"
  export CPPFLAGS="-I/usr/local/opt/node@12/include"

==> Summary
  /usr/local/Cellar/node@12/12.22.8: 3,899 files, 51.5MB, built in 280 minutes 39 seconds
==> Running `brew cleanup node@12`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
{% endhighlight %}

Once we have the chosen formula built (could take several hours!), we ensure that it won't be accidentally overwritten by pinning it:

{% highlight bash %}
$ brew pin node@12
{% endhighlight %}

Should we needed to install a newer version, we'll unpin that version and follow the aforementioned procedure when we have enough time to compile.

For the time being, it only happened to me with Python, Node.js and two or three libraries. In my humble opinion, it's worth it.
