---
layout: post
title: Twisting Python and FreeSWITCH
location: Waterloo, Canada
comments: true
tags:
 - python
 - twisted
 - freeswitch
 - event socket
 - pocketsphinx
 - akinator
---

<style type="text/css">
  .gist-data { height: 250px; overflow: scroll; }
</style>

Do you know [Akinator](http://akinator.com)?

[from wikipedia:](http://en.wikipedia.org/wiki/Akinator)

> Akinator, the Web Genius is an internet game based on Twenty Questions that
> can determine which character the player is thinking of by asking him or her
> a series of questions. It is an Artificial Intelligence program that can find
> and learn the best questions to be asked to the player.

Back in 2008 I was very busy writing web crawlers on python and twisted, as
you can see in [my old blog](http://fiorix.wordpress.com/?s=crawler), which,
although is in portuguese, you'll probably recognize words like *crawler* in
the titles, dates, and code.

By that time I was still learning twisted, and all the black magic of evented
clients and servers in python. Mixing that with [NLTK](http://nltk.org) and a
decent HTML parser like
[Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/) was key to
success.

On spare time, I was getting rid of all my asterisk servers in favor of
freeswitch, and migrating a bunch of dial plans with AGI scripts to
non-blocking [event socket](http://wiki.freeswitch.org/wiki/Event_Socket)
clients and servers in python and twisted.

A year later, already in the telecom business, a challenge came up. If you're
not in this business you may not know it, but, most telecoms are always
looking for new products, and they always end up with yet another creative
of *selling minutes*. This is their karma.

It was no different with me. The challenge was to build a simple POC system,
that would take calls and interact with a web site over the phone. That would
make people call us, bringing in minutes. $$$

What could be better than giving voice to Akinator? No shit. I immediately
connected the dots, and in a few minutes built an interactive web crawler to
[akinator.com](http://akinator.com).

First version was a horrible code, but (as most of what I do :p) worked
flawlessly. It was so perfect that it still works. Following is an updated,
better version of it:

<script src="https://gist.github.com/3152830.js?file=akinator.py"> </script>

If you look at the `interactive` function above, you'll notice it's the
baseline to integrate *any* twisted-based app with this crawler,
assynchronously.

Check this out:

    $ python akinator.py Alex 32 M
    Think about a real or fictional character. I will try to guess who it is.

    Answer: [y]es, [n]o, [?] don't know, [+] probably, [-] not really

    Is your character a male? y
    Does your character live in America? ?
    Is your character real? y
    Is your character currently more than 40 years old? n
    Is your character a singer, or does he work with a singer (as a songwriter, producer, musician...)? n
    Is your character an actor? n
    Is your character linked with sports? y
    Is your character a football (soccer) player? n
    Is your character dark-skinned? y
    Is your character brutal and heartless? y
    Is your character Brazilian? y
    Is your character white? n
    I think of ... Anderson Silva

It just works.

What else did I need to give it a voice? A text-to-speech system. Kid, let me
tell you... not so long ago, getting a TTS system was harder than getting
Internet access in Cuba.

And I desperately needed one for [nuswit](http://nuswit.com) late 2008.
It was a nightmare. I was first ignored by [Nuance](http://nuance.com), then
tried a bunch of other companies, and finally purchased an SDK and some
TTS channel licenses from [Loquendo](http://loquendo.com) - recently acquired
by Nuance!

By the time, I hacked out a `ttsio` command line tool taking text from
*stdin*, dumping wave to *stdout*, which I used to playback.
That was before the existence of
[mod_tts_commandline](http://wiki.freeswitch.org/wiki/Mod_tts_commandline) in
freeswitch.

Anyway, I got it all figured out, and build up the event socket server app for
bridging incoming calls with [akinator.com](http://akinator.com) via the
TTS. But all the interaction with the system was via DTMF. Horrible, but
again, worked flawlessly.

Just recently I was messing up with
[mod_pocketsphinx](http://wiki.freeswitch.org/wiki/Mod_pocketsphinx) totally
unrelated to this, and I needed something to test. The akinator thing came
back to me, and I updated the server code to support it.

All the source code is now on GitHub, with some comments. Go play:

<http://github.com/fiorix/talkinator>

More specifically, this is the server code:

<http://github.com/fiorix/talkinator/blob/master/talkinator/server.py>

This time I made it available in english, on the regular PSTN. With a SIP
account from [freephoneline.ca](http://freephoneline.ca), on this number:

**+1 (226) 336-2189**

It still uses the same TTS system and is a bit slow because it is hosted in
an old server in Brazil. Don't complain.

Enjoy!
