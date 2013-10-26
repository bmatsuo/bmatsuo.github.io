---
layout: post
title: "Concurrency and Responsible API Design in Go"
date: 2013-10-20 14:12:56 -0700
categories: go concurrency
---
[1]: "http://blog.golang.org/advanced-go-concurrency-patterns"
[2]: "http://akka.io/"

##Go + Concurrency = &hearts;

If you haven't seen Sameer Ajmani's talk, [Advanced Concurrency Patterns in Go][1],
watch it. I can wait.

Ok. I hope you enjoyed it. It's an excellent example of how to use Go's
concurrency primitives to serialize access to mutable state. During the Q & A,
Ajmani admits that the concept he present is really a simplication and
specialization of the actor model. People might say that it's weak because
it doesn't provide all the benifits of actor systems like [Akka][]. That's
OK in my opinion. Go's programming model is all about pragmatism. Build a simple
kernel that you can expand upon, layer, or swap *as needed* to optimize your
system. Proper error handling, which the Go language helps enforce, make these
little components much less likely to explode than a fundamentally exception
driven environment like Akka/Scala (although they try to hide it). Furthermore
typesafe channels of communication (which are only exprimental in Akka) mean
that unhandled messages can generally be avoided. Ajmani didn't explicitly
say it. But, his talk shows how these properties can be achieved to a large
degree.

##API design

API design is of utmost importance when designing a package/system. It shapes
the mental model of users and imposes constraints on the designs of their
systems.

##Handling errors

Error handling is a cornerstone of the Go language. Errors must be visible and
explicity handled locally. Often, 'handling' an error means to passing it off to
a higher authority.

This is one area where I disagree with 

##The Gopher Model of Concurrency
