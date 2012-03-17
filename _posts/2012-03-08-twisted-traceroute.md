---
layout: post
title: Twisted traceroute
location: Waterloo, Canada.
tags:
 - python
 - twisted
 - traceroute
 - geoip
 - raw socket
---

The other day I found an old python code lost in a backup, and decided to
turn it into a traceroute utility, on top of Twisted.

In addition to normal traceroute features, it does GeoIP lookups on
 hops' IPs, using [freegeoip.net](http://freegeoip.net).

    $ sudo python txtraceroute.py -n 4.2.2.2
    01. 0.001s :: 173.20y.x.x :: United States, Texas, San Antonio
    02. 0.000s :: 98.12y.x.x :: United States, Texas, San Antonio
    03. 0.001s :: 174.14y.x.x :: United States, Texas, San Antonio
    04. 0.001s :: 4.59.x.x :: United States
    05. 0.002s :: 4.69.x.x :: United States
    06. 0.002s :: 4.2.2.2 :: United States

There are some quite interesting aspects of a python-based traceroute, when it
comes to manually building and parsing IP and ICMP headers with the `struct`
module. Plus the tricky code for ICMP checksum.

On the twisted side, it's all about having a custom raw socket being handled
by the reactor, up with the traceroute requirements of sending synchronized,
 sequential probe packets.

At the end of the day, it's a nice piece of code, which should work just fine
 if you copy and paste into any twisted-based program.

<!--
<script src="http://gist-it.appspot.com/github/fiorix/txtraceroute/raw/master/txtraceroute.py"> </script>
-->

Check out the git repo: [https://github.com/fiorix/txtraceroute](https://github.com/fiorix/txtraceroute)
