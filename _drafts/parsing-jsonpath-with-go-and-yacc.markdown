---
layout: post
title:  "Parsing JSONPath with Go and Yacc"
date:   2013-06-24 00:55:33 -0700
categories: go explorations
---

I've been working on a command line JSONPath implementation written in Go
([link][go-jsontree/cmd/jsontree]). The internals are all done but I need
a parser for JSONPath selectors.

In general Yacc is probably not ideal for parsing JSONPath selectors. I
wrote an effecient scanner but have not put the time in to writing a parser.
I've wanted to at least try Go's yacc tool for some time. So I figured a
JSONPath parser would be a nice refresher and give me a benchmark for when
I write a hand-tuned parser.

##Getting started

[go-jsontree/cmd/jsontree]: "https://github.com/bmatsuo/go-jsontree/cmd/jsonpath"
