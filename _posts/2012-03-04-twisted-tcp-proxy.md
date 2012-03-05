---
layout: post
title: Twisted TCP proxy
location: Waterloo, Canada.
tags:
 - python
 - twisted
 - tcp proxy
---

This is how I built a TCP proxy with Python and
 [Twisted](http://twistedmatrix.com/).

![TCP Proxy]({{site.prefix}}/img/posts/tcp-proxy.png)

The idea is simple: receive client connections, connect to a peer, and
tie them up in a bridge-like mode.

If the peer disconnects, incoming messages are stored by the proxy, until the
connection is reestablished. Once reconnected, all messages are flushed to
the peer.

<script src="https://gist.github.com/1878983.js?file=tcp-proxy.py"> </script>
