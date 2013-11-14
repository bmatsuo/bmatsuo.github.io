---
layout: post
title: "Playing with Camlistore"
date: 2013-11-12 22:36:54 -0800
categories: camlistore tls
---
I'm setting up a `camlistored` server to store photos, my blog, and other data.
[Camlistore](http://camlistore.org/) is an open source "personal storage system
for life".  It's basically a dump for any kind of data with versitile backend
storage options, potentially boundless. Pretty cool stuff.

The heart of Camlistore is a content addressible blob store. The `camlistored`
server indexes structured data stored as JSON blobs. The strategy is quite
similar to Git (mod JSON), perhaps somewhat of an abstraction. I find this
really interesting because I have spent some time working on ways to use Git as
generic datastore. But while built in versioning and free hosting are big
positives for Git as a storage system, I found myself not really needing the
extra functionality of branches, and found them cumbersome when I tried to use
them as an indexing mechanism. It is also hard for tooling to deal with merge
conflicts. This problem is pervasive given Git's distributed nature. The other
end of free hosting cuts deep.

So here we go. Camlistore. The server is self-hosted, but the data can be hosted
out of s3. I can live with that. Although hosting is paid, I have more control
over my data. A monolithic server backed by a transactional index is somewhat
convenient compared to the Git model because merge conflicts don't exist. Load
shouldn't become a problem in a monolithic architecture because the user pool is
just me.

I'm really excited to start tooling around, camli style. I'll be posting about
my experiences in the near future
