---
layout: post
title: "Damn I Love Docker: Plex Media Server"
date: 2018-11-23
category: [docker, containers, devops, rpi]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Plex Media Server](#plex)
* [Installing Plex](#install)
  * [AMD64](#amd)
  * [armhf64](#armhf)
* [Running Plex](#run)
* [Accessing Plex](#access)
* [Initial Configurations](#initconfig)
* [Advanced Configurations](#advconfig)
* [References](#references)

## <a name="install"></a> [Installing Plex](#toc)
In the below sections, you will find the *architecture-specific* commands for
creating a docker container of the *LinuxServer.io*[^fn1] Plex Media
Server.[^fn2] Essentially the only differences between these two commands is the
docker image they are using (i.e. `linuxserver/plex` and `lsioarmhf/plex` for
AMD64 and armhf respectively).

Before looking at the command, it is useful to clarify and briefly explain some
of the options and variables passed to the `docker create`[^fn3] command:

* `--net=host` - Shares host networking with container, **required**.
* `-v /config` - Plex library location. *This can grow very large, 50gb+ is likely for a large collection.*
* `-v /data/xyz` - Media goes here. Add as many as needed e.g. `/data/movies`, `/data/tv`, etc.
* `-e VERSION=latest` - Set whether to update plex or not - see Setting up application section.
* `-e PGID=` for for GroupID - see below for explanation
* `-e PUID=` for for UserID - see below for explanation
* `-e TZ` - for timezone information *eg Europe/London, Asia/Shanghai, etc*

Sometimes when using data volumes (`-v` flags) permissions issues can arise
between the host OS and the container. This can be avoided by specifying the
user PUID and group PGID. Ensure the data volume directory on the host is owned
by the same user you specify and it will "just work."

To find yours use id user as below:

```
$ id $USER
    uid=1001(dockeruser) gid=1001(dockergroup) groups=1001(dockergroup)
```
So from this information, we can see that the `uid=1001`, and the `gid=1001`.
Hence when we run the `docker create` command below, we will set `PUID=1001` and
`PGID=1001`.

### <a name="amd"></a> [AMD64](#toc)
What follows is the docker command for pulling the *LinuxServer.io*[^fn1]
docker image and creating a container to run the *Plex Media Server*[^fn2] on a
typical x86_64[^fn4] CPU architecture
(i.e. your typical PC cpu architecture).[^fn5]
```
docker create \
--name=plex \
--net=host \
-e VERSION=latest \
-e PUID=<UID> -e PGID=<GID> \
-e TZ=<timezone> \
-v </path/to/library>:/config \
-v <path/to/tvseries>:/data/tvshows \
-v </path/to/movies>:/data/movies \
linuxserver/plex
```

As an example we may create a container to run on our *Ubuntu* server from an
earlier post[^fn6] on the Gnosis ML Server:
```
docker create \
--name=plex \
--net=host \
-e VERSION=latest \
-e PUID=1000 -e PGID=1000 \
-e TZ=Asia/Shanghai \
-v /srv/plex:/config \
-v /mnt/EXTDRIVE/movies:/data/movies \
linuxserver/plex
```

Notice here that we have `PUID=1000` and `PGID=1000`, `TZ=Asia/Shanghai`,
`/srv/plex:/config`, and `mnt/EXTDRIVE/movies:/data/movies`. Basically, user
has set the timezone to `Asia/Shanghai`, is storing their *plex config dir* in
`/srv/plex`, and has all of their media on an external drive at
`/mnt/EXTDRIVE/movies`.

### <a name="armhf"></a> [ARMhf](#toc)
What follows is the docker command for pulling the *LinuxServer.io*[^fn1] docker image
and creating a container to run the *Plex Media Server*[^fn2] on a typical
armhf[^fn7] CPU architecture
(i.e. your typical Raspberry Pi cpu architecture).[^fn8]
```
docker create \
--name=plex \
--net=host \
-e PUID=<UID> -e PGID=<GID> \
-v </path/to/library>:/config \
-v <path/to/tvseries>:/data/tvshows \
-v </path/to/movies>:/data/movies \
lsioarmhf/plex
```

## <a name="run"></a> [Running Plex](#toc)
Now that our *architecture-specific* docker container has been created, we can
simply run it with:
```
$ docker start plex
```
This will start the container, and run it as a daemon (i.e. in the background).

## <a name="access"></a> [Accessing Plex](#toc)
Now that the docker container for plex has been built and is running on your
server, you can access it. The most basic way to access the plex application is
through the **WebUI** by navigating in your browser to the following URL:
```
HOST_IP_ADDRESS:32400/web
```
This could be something like:
```
raspberrypi.local:32400/web
```
Or maybe:
```
gnosis.local:32400/web
```
Or simply use the local IP address of the host:
```
192.168.10.20:32400/web
```

## <a name="references"></a> [References](#toc)
[^fn1]: [LinuxServer.io](https://www.linuxserver.io/)
[^fn2]: [Plex Media Server](https://en.wikipedia.org/wiki/Plex_(software))
[^fn3]: [Docker Create](https://docs.docker.com/engine/reference/commandline/create/)
[^fn4]: [x86_64 Wikipedia](https://en.wikipedia.org/wiki/X86-64)
[^fn5]: [Dockerized Plex AMD64](https://github.com/linuxserver/docker-plex)
[^fn6]: [Gnosis ML Server]({{site.baseurl}}/2017/11/01/gnosis/)
[^fn7]: [ARM Wikipedia](https://en.wikipedia.org/wiki/ARM_architecture)
[^fn8]: [Dockerized Plex ARMhf](https://github.com/linuxserver/docker-plex-armhf)
[^fn9]: [Plex Basic Setup](https://support.plex.tv/articles/200288896-basic-setup-wizard/)
