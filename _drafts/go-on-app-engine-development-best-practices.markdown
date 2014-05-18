---
layout: post
title: "Go on App Engine: Development Best Practices"
date: 2014-05-18 02:59:57 -0700
categories: go appengine
---
The Go SDK for Google's App Engine has some quirks to it.  And as far as this
gopher knows, there's no standard way to develop for App Engine.

###Setting it up

Different ideas

- Download/Unpack `go_appengine` somewhere. It should be at the end of your PATH in PATH.

- Download/Unpack `go_appengine` somewhere. It should be at the beginning of your PATH.

- Put `go_appengine` in the project directory. Set PATH, GOPATH.

###Problems

- Getting vim to see packages.
- Vendered init() problems.
- Import path problems.
- http.Client/.Transport exposure in pacakges.

###Sites using the Go SDK for App Engine

I don't know of any.
