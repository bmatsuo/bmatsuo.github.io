---
layout: post
title: "Panic as a control structure: the _nil panic_"
date: 2013-12-11 02:04:58 -0800
categories: thoughts go 
---
A runtime `panic()` is not a common thing in Go. It's pretty amazing. But people
do attempt to use them occasionally. I generally think they're doing it wrong.
Stil, `panic()` exists. I had a fun idea when trying to solve a problem
recently. A 'structured' panic wrapper that is easy to use and safe. Whether or
not it is a good thing is debatable. The core concept is simple.

{% highlight go %}
panic(nil)
{% endhighlight %}

People should be asking "Why the fuck would you panic with a `nil`?". If a
runtime panic bottoms out then it should probably have some information to
say why it blew up, not `nil`. In a perfect world, we could then assume that
any panic with a nil value was meant as flow control.

I don't like when I see people call `panic()`. Newbies think they should be used
like exceptions in other languages. That's OK. They will learn. Other people
like to use them as, to quote my co-worker, "expections". Never meant to leak to
the top level but used as a short-circuiting mechanism. I guess it's better. But
I don't want panics reaching the bottom of the stack ever. And my feeling is
that this kind of use make that much more likely.

##An example

It's basically the problem of middleware. I want to execute an ordered set of
*setup* tasks
then, after processing, a corresponding set of *cleanup* tasks in reverse order.
