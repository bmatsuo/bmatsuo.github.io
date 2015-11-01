---
layout: post
title: "long running commands in linux"
date: 2014-09-06 04:49:43 -0700
categories: linux bash
---
TL;DR -- `(firefox 2>&1 | logger -t firefox -t local0.warn) &`

Having recently switched back to Openbox after many years, and lacking any kind
of configuration, I've been having some isuses launching some programs.  Things
like VLC or something else I download will launch writing to stdout/stderr and
I basically sacrifice a terminal tab devoted to soaking up log messages.

Googling the problem lead to solutions like
[this](http://unix.stackexchange.com/questions/74520/can-i-redirect-output-to-a-log-file-and-background-a-process-at-the-same-time).
I don't really think this is a sustainable, scalable solution though.  Programs
may be running literally for months and rebooting erases the old data.  In the
first case accumulating too much data is a problem and in the second losing
data is the problem.

This is the problem with text logs. So it seems only logical to me to handle
things the same way unix has done for some time, syslog.  The `logger` utility
is helpful for this.
