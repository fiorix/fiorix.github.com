---
layout: post
title: A victim of recursion
location: Philadelphia, USA
comments: true
tags:
 - python
 - twisted
 - LineReceiver
 - recursion bug
 - cyclone
 - txredisapi
---

About a month ago, this [ticket](https://github.com/fiorix/txredisapi/pull/21)
popped up on my non-blocking python redis driver
[txredisapi](https://github.com/fiorix/txredisapi/), about it's inability
to properly parse nested multi-bulk replies from redis server.

Usually, these nested multi-bulk shows up when multi-bulk replies are part
of transactions. Think of an `mget` within a transaction. Its response is
usually a multi-bulk, but because it's a transaction, and the transaction
response is a multi-bulk itself, it becomes a nested multi-bulk.

Pretty fucking complicated, I know... and that's just how the story begins.

Previously, we had an embedded partial parser in the parser, just for the
multi-bulk, but that (obviously!) didn't work for the nested case above.

It took me about 2 days just to realize wtf was going on, but I came up with
a pretty decent and flawless implementation. Here's the
[diff](https://github.com/fiorix/txredisapi/commit/6e0f61dba1105809eb4fa67ce8cb9523748038b8).

However, one of the things I noticed after implementing the perfect, flawless
new version of the parser, is that it was hitting python's maximum recursion
depth. How perfect, eh?

A smart ass latino like me would never give up. So, I just told myself:
the parser is good, and as generic as it should be. This recursion thing is
not my fault, it's either python's or twisted's. Not mine.

It turns out other people tried fixing it before. It's actually a
[known bug](http://twistedmatrix.com/trac/ticket/3050) in twisted's
LineReceiver protocol, and apparently others gave up. Not me.

So, guess what? A 1-liner would do it. The issue happens when we switch
back and forth from line mode to raw mode in LineReceiver. I just used
the reactor itself to call setLineMode and it *apparently* did fix it.

It passed all test cases. Come on,
[we have a bunch](https://github.com/fiorix/txredisapi/tree/master/tests) of
test cases, and if it pass all them and we see then green light from
[travis-ci](http://travis-ci.org), it means it's working.

Except that synthetic test cases always pass anyway.

Three weeks later, I updated [cyclone](https://github.com/fiorix/cyclone), and
updated the redis driver shipped with it. The day after I got a new ticket:
[Redis issues](https://github.com/fiorix/cyclone/issues/63).

Broken parser. Life ain't easy, kid. Using the reactor to switch from raw mode
to line mode didn't work, and completely messed up with the internal buffer
of LineReceiver. It turns out there might be data coming in in between the
call to setLineMode by the reactor, and things get unaligned, and the parser
fails miserably. So bad that it dumps raw redis messages as part of the
responses. Ugly.

When I first looked at the issue, I couldn't believe it. I knew it would be
a nightmare to get that going. [Others](https://github.com/deldotdr/txRedis)
never did, and they chose not to use LineReceiver at all - but they don't
support nested multi-bulk as well, so who cares.

That was 5 days ago. I couldn't stop for a second to think about it... I'm in
Philly since Monday to fix other people's problems. Comcast.

Now, I was just here in my hotel room, reading
[JPL Coding Standard](http://lars-lab.jpl.nasa.gov/JPL_Coding_Standard_C.pdf),
because it was on hacker news earlier this week. Look at *Rule 4*:

> There ***shall*** be no direct or indirect use of recursive function
> calls. &#91;MISRA-C:2004 Rule 16.2; Power of Ten Rule 1&#93;

Recursive function calls are not welcome. Learn that from NASA, twisted.

I immediately went back to the code, and got it working in 15 minutes. With my
own version of LineReceiver, and
[minimal changes](https://github.com/fiorix/txredisapi/commit/5f9e76e71462d84943dd9ee9892b31d54a07b422)
to it. Like it or not, I don't care. And I'm not giving up.

Seriously, simple changes: lines 162 and 163. Then, from 125 to 133. Whatever
happens in between self.pauseProducing() and reactor.callLater(...) won't
mess up with the buffer. Easy.

[All test cases pass](http://travis-ci.org/#!/fiorix/txredisapi/builds/2064145),
but does that mean anything?
