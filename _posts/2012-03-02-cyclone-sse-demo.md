---
layout: post
title: cyclone sse demo
location: Waterloo, Canada
tags:
 - python
 - twisted web
 - cyclone
 - sse
---

A while ago, [@gleicon](http://github.com/gleicon) implemented [server sent
 events](http://en.wikipedia.org/wiki/Server-sent_events) in
 [cyclone](https://github.com/fiorix/cyclone).

During the last code synch up between tornado and cyclone, I put together a nice
[demo app for sse](https://github.com/fiorix/cyclone/tree/master/demos/sse).

Basically, it keeps a single server-side telnet connection with
`towel.blinkenlights.nl` and broadcasts its messages to all browsers listening
to the sse endpoint.

There's a live demo at [http://cyclone.io/ssedemo](http://cyclone.io/ssedemo).
