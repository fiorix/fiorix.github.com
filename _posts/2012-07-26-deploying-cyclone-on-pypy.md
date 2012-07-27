---
layout: post
title: Deploying cyclone on pypy
location: Waterloo, Canada
comments: true
tags:
 - python
 - twisted
 - cyclone
 - pypy
---

Tired of typing the same thing all over. With all the differences on osx,
debian 32, and 64 bits. Different pypy versions, etc... ended up with a couple
of bash scripts to do it for me.

One of them installs pypy 1.9 on /opt and make all symlinks to
/usr/local/bin, with pip and virtualenv. The other installs pyopenssl,
twisted and cyclone, and make the symlinks as well. All links on
/usr/local/bin are prefixed with pypy_, like pypy_pip and pypy_twistd.

That's the easiest and simplest generic way I found to install it:

    curl cyclone.io/install-pypy.sh | bash
    curl cyclone.io/install-pypy-cyclone.sh | bash

And when I deploy my cyclone apps, it's easy to edit the rc.d init file and
point it to use pypy_twistd.
