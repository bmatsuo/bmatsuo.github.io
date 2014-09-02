---
layout: post
title: "The Docker desktop"
date: 2014-09-02 03:09:17 -0700
categories: linux docker home
---
The popular devops community is definitely excited about the potential of
Docker for service deployment and isolation.  It is a stretch for me to claim
membership of that community.  But still, I am excited about using Docker as
well.  In my experimentation trying to become comfortable with a Docker-based
development cycle, which I am still not completely comfortable with, I have
been pondering another question.  Is Docker fit for isolation and packaging in
the Linux desktop ecosystem?

Docker appears to be fairly easy to use with popular, modern init systems like
[upstart and systemd](https://docs.docker.com/articles/host_integration/).  But
in my experimentation with a few simple services,
[Nginx](https://registry.hub.docker.com/_/nginx/) and
[Syncthing](https://registry.hub.docker.com/u/ncadou/syncthing/), I found some
troubling patterns that would make a general deployment of a Docker-based
desktop difficult.

##Bind mounts

In both of my example use cases I found myself wanting to bind-mount
configuration and data directories to the host machine.  This is a pretty
personal task.  Syncthing is the best example of the problem with bind mounts.

Syncthing may operate on any number of directories across volumes.  The
cleanest way to do this is to mount each repository into the home directory of
syncthing in the container.  Then the syncthing web UI (or API) could then add
the folder as a repository as normal.

The `docker run` interface for bind mounts is not completely unusable.  It is
possible to modify the upstart script, destroy, rebuild, and restart the docker
container.  But I think the ideal thing would be a separate text file with a
list of mount locations. Of course automated/assisted reconstruction of the
container would be required.  And this would lead to a cascade effect on linked
containers, if any exist.

I am not aware of any tooling for manipulating a configured set of bind mounts.
I would love to find such a thing.

##Communication and linking

Container linking seems like a tricky thing.  Say I install the Syncthing
container.  Now I need to link my Nginx container to proxy port 8080 in
Syncthing.  Further, if syncthing is running on the host machine there seems to
be no way to 'link' the host to the container.  Anyway, maintaining a setup
like this seems like a fucking nightmare.

I'm not sure if docker containers can be configured to use the system DBus.  Or
if the docker daemon exposes enough information through DBus to implement
container-service discovery.  That may not be the right discovery model though.
I don't know.  Web/Socket interfaces seem more prevalent and portable these
days to me.

In my mind a very natural sounding solution is to run a discover service like
[coreos/etcd](https://github.com/coreos/etcd) or
[hashicord/consul](https://github.com/hashicorp/consul) inside a container, in
the docker [host subnet](https://docs.docker.com/articles/networking/).
Orchestrating disparate services like Syncthing and Ngninx containers can be
handling with configuration shim like
[kelseyhightower/confd](https://github.com/kelseyhightower/confd) deployed
alongside each service.  Future applications can be designed around services
such as etcd and consul.  They will have no need for a shim inside the
container.

I haven't seriously played with confd yet.  But it sounds like a solution to
the problem with nginx.  When an HTTP service begins publishing its presence to
etcd, confd can write a 'proxy_pass` nginx site description for the service and
reload nginx.  Easy peasy.

It seems to me that if confd can satisfy my nginx task and creating the
syncthing container is not too difficult (it has to advertise syncthing to
etcd) I don't see why this pattern wouldn't scale to any number of desktop
services.

I think that
[GoogleCloudPlatform/kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)
is designed to tackle a similar problem although it is most definitely aimed at
large-scale server deployments.  It might be interesting to see the idea
applied to single-instance desktop machines.  Maybe in the future there is a
mixture of local and 'cloud' deployments for a single system.  Again, Syncthing
being a great example, there are times where a cloud extends outside of bare
metal desktops and into virtual cloud infrastructure.  Syncthing daemons
running on cloud instances help provide availability, especially for in small
clusters (connected to a small number of correlated power sources).  As prices
of cloud compute and storage continue to drop, and internet security improves,
this may become a much more viable option for "personal infrastructure" in the
future.
