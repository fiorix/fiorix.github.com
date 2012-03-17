---
layout: post
title: UNIX sockets in txredisapi
location: Mexico City
tags:
 - python
 - twisted
 - redis
 - txredisapi
---

A couple of days ago I implemented UNIX domain sockets client support in [txredisapi](https://github.com/fiorix/txredisapi/).
It took me less than an hour [to do so](https://github.com/fiorix/txredisapi/commit/28f9689843ba00966e1e6c9af48fb177739eee12).

These are the new connection methods:

<pre class="prettyprint">
UnixConnection(path="/tmp/redis.sock", dbid=0, reconnect=True)
</pre>

Connect to a redis server.

`path` is the path of the unix socket where the redis server is listening on.

`dbid` is the server's database ID.

`reconnect` tells the driver to automatically reconnect to the server if
the connection is interrupted during operation, due to a redis restart,
for instance.


<pre class="prettyprint">
UnixConnectionPool(path="/tmp/redis.sock", dbid=0, poolsize=10, reconnect=True)
</pre>

Same as above, but creates a connection pool.

Requests are distributted across all connections in a round-robin manner,
except for transactions. During transactions, all commands are executed
on the same connection. Such connection is excluded from the pool until
the transaction is either commited or discarded.

<pre class="prettyprint">
ShardedUnixConnection(paths, dbid=0, reconnect=True)
</pre>

Connect to multiple redis server instances, and automatically distribute
the requests across all of them using a consistent hashing algorithm on
the request's key value, or values, in case or *mset*, *mget* and alike.

Each connection is treated as a shard.

`paths` is a python list, like `["/path/to/redis1.sock", "redis2.sock"]`


<pre class="prettyprint">
ShardedUnixConnectionPool(paths, dbid=0, poolsize=10, reconnect=True)
</pre>

Same as above, but creates a connection pool for each shard.

<hr>

All connection methods above return a ``deferred`` object, which is fired with a
``UnixConnectionHandler`` when the connection is established.

Example:

<pre class="prettyprint linenums">
def conn_cb(redis):
  redis.set("foo", "bar")

def main():
  d = txredisapi.UnixConnection()
  d.addCallback(conn_cb)
</pre>

Or, the way we suggest (much better), using Twisted's [inline callbacks](http://twistedmatrix.com/documents/12.0.0/api/twisted.internet.defer.html#inlineCallbacks):

<pre class="prettyprint linenums">
@defer.inlineCallbacks
def main():
  redis = yield txredisapi.UnixConnection()
  yield redis.set("foo", "bar")
</pre>


The ``UnixConnectionHandler`` object is the client interface with redis.
It accepts pretty much all redis commands such as *get*, *set*, *count*, etc.

All redis commands return a deferred, which is fired when the command is
accepted by the server (like *set*), or when the server returns
data (like *get*).

If it's disconnected, the *errBack* of the deferred is called instead.

Example:

<pre class="prettyprint linenums">
@defer.inlineCallbacks
def main():
  redis = yield txredisapi.UnixRedisConnection()
  ...
  try:
    val = yield redis.get("key")
  except txredisapi.ConnectionError:
    print("redis is unavailable, try again later.")
</pre>


All the *lazy* versions for these connection methods are available too:

<pre class="prettyprint">
lazyUnixConnection(path="/tmp/redis.sock", dbid=0, reconnect=True)
lazyUnixConnectionPool(path="/tmp/redis.sock", dbid=0, poolsize=10, reconnect=True)
lazyShardedUnixConnection(paths, dbid=0, reconnect=True)
lazyShardedUnixConnectionPool(paths, dbid=0, poolsize=10, reconnect=True)
</pre>

For those not familiar with the the *lazy* connection methods, they return the
``UnixConnectionHandler`` directly.

The idea is that they don't hang when called, and return the handler object
which will eventually connect to redis in background. If the connection drops
anytime, they might reconnect automatically.

These lazy methods are supposed to be used in server applications which will
query the redis server upon request, such as web servers.

<hr>

As expected, the UNIX socket connection is faster than the default INET socket.
This simple benchmark script can measure the difference between the two, and
generally shows about 20% performance improvement over INET sockets.

<script src="https://gist.github.com/2049984.js?file=txredisapi_inet_vs_unix.py"> </script>

For more information, check out the [README](https://github.com/fiorix/txredisapi/blob/master/README.rst) shipped with [txredisapi](https://github.com/fiorix/txredisapi).
