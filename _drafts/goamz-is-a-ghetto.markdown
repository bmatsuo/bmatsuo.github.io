---
layout: post
title: "goamz is a ghetto"
date: 2014-10-03 22:16:28 -0700
categories: aws go
---

I've come to dislike using goamz quite a bit.  It's pushing me to consider
alternative packages.  Unfortunately it is the de facto package and there are
not a lot of options.  Fewer options if you want something with adequate real
world testing.

Goamz very inflexible and it seems that just about every time I do something I
haven't tried before and that the AWS docs indicate is possible I find it's not
fully supported in the package.  Creating patches is very arduous.  Adding
methods to the package types is straight forward but much pain is endured
adding tests consistent with the existing test code.

##These fucking tests

In my experience, specifically dealing with the EC2 package, it is worse
because when writing the tests I find that the relevance of the tests in the
package is very questionable in the grand scheme of things.  And wasting a
shitload of time writing useless tests is something that **really** grinds my
gears.  The tests contain example requests and responses which are parsed and
values are checked in full.  It is effectively a test that the XML struct tags
are "properly" named/typed.  But, Amazon generally gives very few example
request and response bodies for any single API call and rarely do they express
all possible variation in an request/response.  So you basically have to make
stuff up or just don't "test" the missing pieces.

From what I've experienced the APIs provided by AWS all appear to be too
complex for any single developer, and maybe even a small sized organization, to
understand fully and much less test in this draconian scheme.  The inevitable
result is that users routinely find applications which are not handled properly
or are incomplete in the packages.

##This fucking monolith

Packages like @bmizerany's aws4 are interesting because they focus on a single
problem.  In this case, the problem is signing requests.  I'm a little unclear
about the intent behind the repo (and it may be dead).  There is (the beginning
of) a DynamoDB package named dydb.  Blake may have intended for aws4 to be the
next goamz.  But in my opinion that is the wrong approach at this point in
time.

I think there is a problem with monolithic libraries that include packages for
every aws service.  It seems both difficult and completely unnecessary for a
single organization to take on maintainership of packages for all AWS services.
And if signing can be abstracted and generally unified with a common package
like AWS4 it's even easier to split things out into a decentralized AWS package
ecosystem.

I think that ideally organizations that are experts with particular services
can own separate packages specifically for each services, potentially focused
only on aspects of each service.

Going even further, packages may only provide a method for creating http
requests, using something like XPath (@gniemeyer's xmlpath package).  Gustavo
hinted as much in his
[blog](http://blog.labix.org/2013/06/07/efficient-xpath-for-go), posted
sometime back.

XPath is interesting.  As Gustavo shows, it can be quite efficient for simple
queries into XML structures.  And this removes any question of testing response
structures (except, possibly for error structures).  The really tempting aspect
is that response handling is more flexible (new API versions less likely to
require a package upgrade). And when package upgrades are required implementing
APIs and API actions is simplified significantly.

There is always the potential of generating response structures from XSD
documents (I think Amazon has one somewhere).  But I don't think people have
put a lot of work into that.  That is it's own problem because understanding
XML Schema to an adequate level to generate Go structures properly is not
trivial.

##This fucking signature

Anyway, I've been talking about aws4 as if it's going to save the world
(despite being potentially dead).  But when I tried to achieve something like I
mention above using aws4 I was confronted with a problem I had no idea how to
solve properly.

The AWS docs for signature version 4 are not unclear.  And aws4 appears to be
implemented correctly if not completely (no signed urls iirc).  But when I
attempted to implement signed URLs and use aws4 with the S3 API I found that,
while it proports to use signature version 4, it does not use the exact
algorithm described in the actual AWS signature version 4 specification
document.  If I recall correctly the difference is in the signing of the PUT
request entity content.  It wasn't an insurmountable obstacle.  But I couldn't
help but being left with a feeling I couldn't shake.  The feeling that other
APIs would encounter other bizarre and subtle inconsistencies.  At some point I
imagine the result would be a code mess given the design of aws4.

Interfaces feel like a good way to deal with the problems of inconsistency.
And I think someone should attempt should a modification of aws4 sometime in
the future.
