---
layout: post
title: Twisted TCP proxy
location: Waterloo, Canada
comments: true
tags:
 - python
 - twisted
 - tcp proxy
---

This is how I built a TCP proxy with Python and
 [Twisted](http://twistedmatrix.com/).

![TCP Proxy]({{site.prefix}}/img/posts/tcp-proxy.png)

The idea is simple: take client connections, connect to a peer. Tie them up in
 a bridge-like mode, by binding their events.

In this example, the read event from the client turns into a write event to
 the peer, and vice-versa. Client disconnection terminates the session.

But the peer connection is persistent. If the peer disconnects during a session,
the proxy stores client messages until the peer connection is re-established.
Wouldn't be a big deal to turn this into a tcp proxy and load-balancer, or a
 tcp pubsub server.

A simple combination of the [Reconnecting Client Factory](http://twistedmatrix.com/documents/current/api/twisted.internet.protocol.ReconnectingClientFactory.html)
 and a pair of [Deferred Queues](http://twistedmatrix.com/documents/current/api/twisted.internet.defer.DeferredQueue.html) do the job.

<script src="https://gist.github.com/1878983.js?file=tcp-proxy.py"> </script>
