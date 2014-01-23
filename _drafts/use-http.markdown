---
layout: post
title: "Use HTTP"
date: 2013-11-30 15:24:52 -0800
categories: http
---
REST is dead. I am talking about the term. It lost any meaning it had because
people use it so damned loosely. When people talk about REST, the are almost
assuredly talking about *routes* (HTTP service endpoint structure). This is
primarily because "RESTful routes" is so much fun to say. And then, when a
company says they have a RESTful API (that is, RESTful routes). All too often
it becomes clear that a companies APIs are not particularly RESTful for some
reason or another, and the term was just slapped on as a buzzword. So no one
knows what REST is. Fuck. Well here's an idea.

##Learn and use HTTP

Many ideas in REST, as far as web services are concerned, simply boil down to
using the facilities HTTP provides instead of reinventing them. Something I see
often is overloading a subset of the protocol with semantics that HTTP provides.

###Use methods

Why have service endpoints with distinct URLs

    GET  http://example.com/myresource
    POST http://example.com/myresource/replace
    POST http://example.com/myresource/update
    POST http://example.com/myresource/delete

. You can use the HTTP [methods](#) to handle the semantics and use the same URL
(the location of the resource! zomfg!) for all the endpoints

    GET    http://example.com/myresource
    POST   http://example.com/myresource
    PUT    http://example.com/myresource
    DELETE http://example.com/myresource

. HTML 5 allows form elements to use the PUT and DELETE methods natively, and they
are very common in client side javascript frameworks and toolkits like jQuery.

##Use caching

##Use status codes

##Use headers

It's pretty lame by most people's standards, but I've read though the [list](#)
of HTTP 1.1 headers. I didn't read all the technical details of the more nuanced
headers. And I skipped most things related related to proxies (to return again
later).  There's some interesting stuff in there.

On my last look I noticed the `Server` header (14.38). At my job we talked about
injecting git commit ids into response headers we talked a little about the best
name and settled on `Janrain-Server-Version` as something we could use uniformly
across the company. `>_<`

I also noticed that there is a `Warning` header (14.??). This is fantastic. Some
of the codes used are only related to proxies. But servers get codes 199, and
299 to issue whatever warnings they want. I've wondered about a good way to
deliver warnings before. Putting it in the response body always seemed like it
wasn't the best fit because I don't have a warning *class* of responses. They
are successful responses, with extra warning messages about potential problems.
In a language/program that marshals to JSON/XML based on class/struct fields,
inserting extra warning fields kind of sucks.

**Note:** I don't actually know if warnings are displayed in the browser (the
spec language make it sound like they should be displayed in the console).
But I think that the header is useful completely outside of the browser.
And I can use Angular to inspect response headers and build my own warning system
if necessary. So I consider native browser support a bonus.
