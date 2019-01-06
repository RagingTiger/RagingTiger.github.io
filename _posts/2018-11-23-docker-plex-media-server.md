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
  * [AMD64 - PC Install](#amd)
  * [armhf - Raspberry Pi Install](#armhf)
* [Running Plex](#run)
* [Accessing Plex](#access)
* [Initial Configurations](#initconfig)
* [Advanced Configurations](#advconfig)
  * [Backing Up Media and Data](#advconfig)
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

To find yours use `id $USER` as below:

```
$ id $USER
    uid=1001(dockeruser) gid=1001(dockergroup) groups=1001(dockergroup)
```
So from this information, we can see that `uid=1001`, and `gid=1001`.
Hence when we run the `docker create` command below, we will set `PUID=1001` and
`PGID=1001`.

### <a name="amd"></a> [AMD64 - PC Install](#toc)
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
-v /mnt/DATADRIVE/movies:/data/movies \
linuxserver/plex
```

Notice here that we have `PUID=1000` and `PGID=1000`, `TZ=Asia/Shanghai`,
`/srv/plex:/config`, and `mnt/DATADRIVE/movies:/data/movies`. Basically, the
results of the `id $USER` showed `uid=1000` and `gid=1000`, hence the user
set `PUID=1000` and `PGID=1000`, the user set the timezone to `Asia/Shanghai`,
and has created their *Plex config dir* in `/srv/plex` with all of their media
in a drive located at `/mnt/DATADRIVE/movies`.

### <a name="armhf"></a> [ARMhf - Raspberry Pi Install](#toc)
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

**NOTE**: As an example for the Raspberry Pi, we will likely want to consider
using an **external drive** to store the Movies and the Plex configuration
directories. This is due to the more robust I/O capabilities of a Hard Disk
Drive vs. a micro SD card.[^fn9] More information on the formatting process of
the drive can be found in another post[^fn10] devoted to reformatting a hard
disk drive in **ext4**.

Once we have our external drive ready (in this case mounted to `/mnt/PIDRIVE`),
we can create the container as follows:
```
docker create \
--name=plex \
--net=host \
-e VERSION=latest \
-e PUID=1001 \
-e PGID=1001 \
-e TZ='America/Chicago' \
-v /mnt/PIDRIVE/plex_config:/config \
-v /mnt/PIDRIVE/movies:/data/movies \
lsioarmhf/plex
```

Notice here that we have `PUID=1001` and `PGID=1001`, `TZ=America/Chicago`,
`/mnt/PIDRIVE/plex_config:/config`, and `/mnt/PIDRIVE/plex_config:/data/movies`.
Basically, the results of the `id $USER` showed `uid=1001` and `gid=1001`, hence
the user set `PUID=1001` and `PGID=1001`, the user set the timezone to
`America/Chicago`, and has created their *Plex config dir* on their external
drive at `/mnt/PIDRIVE/plex_config` with all of their media on the same external
drive at `/mnt/DATADRIVE/movies`.

## <a name="run"></a> [Running Plex](#toc)
Now that our *architecture-specific* docker container has been created, we can
simply run it with:
```
$ docker start plex
```
This will start the container, and run it as a daemon (i.e. in the background).

## <a name="access"></a> [Accessing Plex](#toc)
Now that the docker container for Plex has been built and is running on your
server, you can access it. The most basic way to access the Plex application is
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

## <a name="initconfig"></a> [Initial Configuration](#toc)
Without going into too much detail, the initial setup will require two parts:
  * 1: You will be prompted to login or create a Plex account
  * 2: You must select where the media is stored that will be used with Plex

Basically after creating a Plex account, or logging in to an existing account
just make sure to add a Movie library, and use the path to `/data/movies` as the
path to where the media are located. More detailed information can be found in
the official Plex post[^fn11]

## <a name="advconfig"></a> [Advanced Configurations](#toc)
In this section we will discuss the various advanced features of both Plex, and
the host operating system for configuring your system for various needs (e.g.
backing up your media, downloading subtitles automatically, etc ...)

### <a name="backup"></a> [Backing Up Media and Data](#toc)
One of the best ways to backup any *Unix-like* system is through a utility
known as `cron`. While we will not discuss in detail how to edit a `crontab`, we
will refer you to a previous post on the subject.

Instead we will show you how you an example of a `crontab` entry that will
backup the external drive containing all your media files and Plex data to
another external drive:
```
$ sudo crontab -e
...
...
...
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,$
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

# backup PIDRIVE to PIBACKUP
01 01 * * * sudo rsync -havP /mnt/PIDRIVE/ /mnt/PIBACKUP/
```

What the above example shows, is that the `rsync` command will run with `super
user` privileges at **01:01 am** or one minute past one in the morning (or late
at night if you are not a morning person). When it runs it will backup the
contents of the drive at `/mnt/PIDRIVE` to `/mnt/PIBACKUP`. This will ensure
that every night at one minute past one in the morning, the drive with all your
Plex data and media, will be copied to the drive at `/mnt/PIBACKUP`


## <a name="references"></a> [References](#toc)
[^fn1]: [LinuxServer.io](https://www.linuxserver.io/)
[^fn2]: [Plex Media Server](https://en.wikipedia.org/wiki/Plex_(software))
[^fn3]: [Docker Create](https://docs.docker.com/engine/reference/commandline/create/)
[^fn4]: [x86_64 Wikipedia](https://en.wikipedia.org/wiki/X86-64)
[^fn5]: [Dockerized Plex AMD64](https://github.com/linuxserver/docker-plex)
[^fn6]: [Gnosis ML Server]({{site.baseurl}}/2017/11/01/gnosis/)
[^fn7]: [ARM Wikipedia](https://en.wikipedia.org/wiki/ARM_architecture)
[^fn8]: [Dockerized Plex ARMhf](https://github.com/linuxserver/docker-plex-armhf)
[^fn9]: [HDD vs. Micro SD Card](https://www.reddit.com/r/raspberry_pi/comments/6uauel/external_hdd_vs_big_sd_card_for_rpi3/)
[^fn10]: [ext4 Reformatting in Linux]({{site.baseurl}}//2018/12/08/format-disk-linux/)
[^fn11]: [Plex Basic Setup](https://support.plex.tv/articles/200288896-basic-setup-wizard/)
