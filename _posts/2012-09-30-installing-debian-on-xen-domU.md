---
layout: post
title: Installing Debian on Xen domU
location: Waterloo, Canada
comments: true
tags:
 - linux
 - debian
 - xen
 - pygrub
 - kvm
 - qemu-nbd
---

Today I learned.

It's been quite some time since I don't do sysadmin work. Today I had to
install a new debian system on a xen vm, and it turns out it's not as
straightforward as it should.

Well, back in the days I was told to install a debian somewhere else, like
vmware, then make an image of it (literally a tarball of /) so I could simply
uncompress on any lvm logical volume and map directly as sda1. That's what
I've been doing since then, until today, when I actually needed a new image.

I started following instructions from this document:
<http://wiki.xen.org/wiki/Debian_Guest_Installation_Using_Debian_Installer>

Downloaded the iso, vmlinuz, initrd.gz and made the necessary changes to
xm-debian.cfg, as the document says.

It didn't work. This pygrub thing seems broken to me... don't get me wrong,
as I do like python myself. But there's something really funky going on with
pygrub, and it's not the first time I get crappy meaningless debug messages
from it.

After hours debugging those *Error: Boot loader didn’t return any data!* and
then *Error: Invalid mode*, I almost gave up, until I found this:
<http://www.howtoforge.com/installing-debian-squeeze-6.0-domu-on-centos-5.5-x86_64-dom0>

The only setup that works for me is setting the debian iso as *xvda* and my
logical volume as *xvdb* on xm-debian.cfg, with this command line:

    xm create -c /etc/xen/xm-debian.cfg install=true \
     install-mirror=ftp://ftp.us.debian.org/debian \
     install-installer=ftp://ftp.us.debian.org/debian/dists/squeeze/main/installer-amd64/20110106+b1/images

After the installation, I managed to mount the logical volume to generate my
new tarball image of the system.

On other systems I have kvm, and mount the disks using qemu-nbd. What it does
is it maps a disk image (again, I use lvm logical volume) into an nbd, so you
have access to its partitions and can mount via /dev/nbd0pX - where X is the
partition number.

With xen, tho, I managed to mount the disk image directly, with instructions
from here:
<http://www.magicspace.eu/linux/direct-mount-xvda-devices-or-multi-partition-isos/>

This is all on a paravirtualized vm - no hvm (vmx/svm).
