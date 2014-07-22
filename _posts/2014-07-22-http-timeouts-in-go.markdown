---
layout: post
title: "HTTP timeouts in Go"
date: 2014-07-21 23:30:44 -0700
categories: go http
---

**TL;DR** The standard Go HTTP client does not directly provide "open" and
"read" timeouts as found in libraries for other languages.  Go 1.3 added the
facility for implementing these types of timeouts indirectly.  This post
describes how to implement these timeouts using
[net.Dialer](http://godoc.org/net#Dialer) and, in general, how the timeout
system in "net/http" works.

###One timeout to rule them all

Prior to Go 1.3, you couldn't find timeouts mentioned in the documentation for
[http.Client](http://godoc.org/net/http#Client). In 1.3, a timeout was added to
the type, but it's a single `Timeout`, representing both "open" and "read"
time.  This is probably good enough in most situations. And it's nice and
simple compared to two timeout approaches.  It's a classic thought in my mind,
"I don't want this to take longer than 2 seconds. Now how long should I let it
connect?". In the past I've made arbitrary decisions how to split that 2
seconds between "open" and "read".  Go just lumps them together the same way my
brain does.

OK. So I just convinced you to not give a shit about distinguishing between the
two types of HTTP timeout, right? I hope not (though I wouldn't blame
completely you).

###A case for "open"

I recently ran into a case where an open timeout was actually important.
Operating with large files on S3. For a very large file (GBs in size) there is
no commercial backbone I'm aware of that could deliver me the data fast enough
to get it in seconds. Maybe crazy Range-fu funneling **lots** of requests into
a single file could get the complete file within a reasonable wall clock time.
For this scenario assume the Range header is not supported.

So, you are resigned to having a very long wall clock timeout on the request.
But what happens if your client is never able to connect to S3? You are waiting
on a dead connection for an hour, that's what.

I think this is generally a good rule of thumb. Slow servers should have their
"connect" and "transfer" timeouts disambiguated.  The only question is how to
do that using the Go standard library.  It turns out to be pretty
straight-forward.

###Separate HTTP client timeouts

The setup for declaring timeouts in Go is the way it's because of how
"net/http" modularizes and separates concerns in the client. There is a
function for dialing (opening connections), a and a function for executing a
round-trip request over a previously dialed connection. The interface at the
client's abstraction is very simple and convenient while still allowing for
things like connection pooling. But the simplicity can make other things (in
this case controlling timeouts other than wall clock time) more difficult to do
correctly or otherwise awkward.

{%highlight go%}
open := time.Second
read := time.Minute
client := &http.Client{
    Dial: (&net.Dialer{
        Timeout: open,
    }).Dial,
    Timeout: open + read,
}
resp, err := client.Get("http://s3.amazonaws.com/example/path/to/some/huge/file.tar.gz")
// ...
{%endhighlight%}

The above solution comes from learning and exploiting concepts the client and
the Go standard library provide.  Dialing is obviously the place where "open"
timeouts apply.  At this point we take a shortcut instead of implementing a
"read" timeout directly, if it's necessary to separate timeouts and the dialer
is explicitly specifying its timeout already, the client's wall clock `Timeout`
field can be used to specify the read timeout (indirectly as the sum "open" +
"read").

It's worth noting that instead of using the `*net.Dialer`, a more straight
forward dial function literal can be constructed using `net.DialTimeout`.  I
prefer the approach I gave because it allows further customization of the dial
behavior. See its [godoc](http://godoc.org/net#Dialer) for details.

###Takeaways

While not directly apparent there is a rich facility for managing the
[http.Client](http://godoc.org/net/http) within the standard library (it's not
only in this case; trust me). The package's design exemplifies the philosophy
that simple, general purpose types and functions should be specialized for
unique situations encountered everyday. The result is a highly flexible HTTP
client that utilizes strict types and scales from the simplest use case to edge
case (at least in this modern with with fast, 'enterprise', 'cloud' APIs
&ast;puke&ast;).

###And here's the package that wraps it all up

No. Fuck no. This is simple enough to repeat as needed. If it compiles, it does
what you are telling it and you only risk not knowing what you really want
(which is a personal problem).
