---
layout: post
title: "Firewalling with iptables or a dalliance with Docker"
date: 2013-10-13 14:53:19 -0700
categories: linux security
---
[1]: http://ipset.netfilter.org/iptables.man.html "man page"
[2]: http://serverfault.com/questions/184646/a-secure-standard-iptables-rule-set-for-a-basic-https-webserver "Server Fault example"
[3]: https://help.ubuntu.com/community/IptablesHowTo "Ubuntu help page"

My web server is running very little software as of writing this.
It runs sshd, nginx, and a stub "hello" go app (plus 'normal' linux stuff,
whatever that means). But I want to start doing real stuff with it. This
generally means running a couple other services such as a database deamon,
redis-server/memcached, etcd, etc (cannot...resist...).

I figure that the easiest thing I can do to secure the machine is to set
up a restrictive firewall. If I do this I can be less strict about securing
communication between components contained entirely within the machine. If/When
the time comes to split some of these service components on to separate logical
machines I will deal with securing the communication of these services on a case
by case basis.

As far as I am aware, the standard way to set up a firewall on a Linux machine
is through *iptables*. The `iptables` command configures the Linux kernel's
(IPv4) packet filter [1][]. So I will begin my quest of practical internet
security there.

##iptables wtf

I am coming from a space of sheer ignorance. I've never used firewalls on home
machines. I used to play a lot of games and I found it a big annoyance. Since
then I've built up a disipline of safe computer use, minimizing exposure to
potential harmful situtations. Still, this is a dumb excuse and really I've
playing it dangerously been unsafe for a long time. This needs to be fixed.

The first thing I did to learn about iptables was google "sercuring a box with
iptables", leading me to a Server Fault [discussion][2]. Oh, fuck, what am I
getting into here. This is a massive shell script. Most of it parses and make
sense.  But there is some stuff I don't grok.
{% highlight bash %}
  # default: DROP!
  $ip -P INPUT DROP
  $ip -P OUTPUT DROP
  $ip -P FORWARD DROP
{% endhighlight %}
Am I dropping all existing connections? I don't see anywhere that things get
committed. So are they taking effect immediately?
{% highlight bash %}
  sshers=""
{% endhighlight %}
This has no ssh priviledge? Do I even want a static list of IPs for ssh?
Do I run the risk of locking myself out entirely? Then there's a whole bunch
of shit that I can't tell whether I need or not.
{% highlight bash %}
  # temporary backup if we change from DROP to ACCEPT policies
  $ip -A INPUT -p tcp -m tcp --dport 1:1024 -j DROP
  $ip -A INPUT -p udp -m udp --dport 1:1024 -j DROP
{% endhighlight %}
Wtf does that mean?

Clearly, I need to learn more of the basics about using iptables.
Cargo culting this would be a huge mistake. Also, with questions about whether
or not I am going to lock myself out of my web server, I need to start on a
virtual machine that I can easily blow away. This sounds like something that
Docker is supposed to do well, so I'll be throwing some docker bits at you
throughout this article.

Anyway, I googled "iptables basic setup" and found an Ubuntu community
documentation [help page][3] that looks right up my alley.
