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
* [Notes On Hardware](#notes)
* [References](#references)

## <a name="install"></a> [Installing Plex](#toc)

### <a name="amd"></a> [AMD64](#toc)
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
-v </path for transcoding>:/transcode \
linuxserver/plex
```

### <a name="armhf"></a> [ARMhf](#toc)
```
docker create \
	--name=plex \
	--net=host \
	-e PUID=<UID> -e PGID=<GID> \
	-v </path/to/library>:/config \
	-v <path/to/tvseries>:/data/tvshows \
	-v </path/to/movies>:/data/movies \
	-v </path for transcoding>:/transcode \
	lsioarmhf/plex
```

## <a name="references"></a> [References](#toc)
[^fn1]: [Dockerized Plex AMD64](https://github.com/linuxserver/docker-plex)
[^fn2]: [Dockerized Plex ARMhf](https://github.com/linuxserver/docker-plex-armhf)
