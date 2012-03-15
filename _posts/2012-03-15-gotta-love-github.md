---
layout: post
title: Gotta love GitHub
location: Mexico City
tags:
 - github
 - jekyll
 - rss
 - twisted
 - mongodb
 - redis
 - mongo-async-python-driver
 - txredisapi
---

<div style="float:left;margin:0 15px 10px 0;" class="thumbnail">
  <img src="{{site.prefix}}/img/posts/mexico-skull.jpg" alt="">
</div>

GitHub is amazing. They host our projects in a very nice way, they provide us
with tools for creating things like this blog, their issue tracker and automatic
merge feature works just fine.

First, I'd like to thanks [@gleicon](http://github.com/gleicon) for taking on
my projects while I'm travelling with very limited Internet access.

Over the week, he's been actively working on the [asynchronous python driver
for mongodb](https://github.com/fiorix/mongo-async-python-driver/), and the
[asynchronous python driver for redis](https://github.com/fiorix/txredisapi/).

By the way, there's very interesting stuff going on with the mongo driver,
like this ticket about [supporting Geo2d and spatial indices](https://github.com/fiorix/mongo-async-python-driver/issues/14).

The automatic merge feature is very, very useful. This morning, even with the
super-slow Internet access at a Starbucks, I could merge a couple of pull requests
with a single click.

Thanks, GitHub, for making it so easy.
Without this feature, it would take ages to pull all the code down to my computer,
merge it, and push it back.

Last, but not least, another thanks to GitHub and Jekyll. It took me about 5
minutes to add RSS support for the entire blog.
It was just a matter of creating an XML template. I grabbed [the RSS feed
template](http://coyled.com/2010/07/05/jekyll-templates-for-atom-rss/)
from coyled.com.

Now, folks who have been asking for it, like [@MarceloToledo](http://twitter.com/#!/marcelotoledo),
can start following right away.
<a href="{{site.prefix}}/rss.xml"><img src="{{site.prefix}}/img/feed-icon-20x20.png"></a>
