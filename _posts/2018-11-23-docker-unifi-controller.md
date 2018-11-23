---
layout: post
title: "Damn I Love Docker: UniFi Controller"
date: 2018-06-19
category: [docker, containers, devops, rpi]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [UnifiController](#plex)
* [Installing UniFi Controller](#install)
  * [AMD64](#amd)
  * [armhf64](#armhf)
* [Notes On Hardware](#notes)
* [References](#references)

## <a name="install"></a> [Installing UniFi Controller](#toc)

### <a name="amd"></a> [AMD64](#toc)
```
docker create \
  --name=unifi \
  -v <path to data>:/config \
  -e PGID=<gid> -e PUID=<uid>  \
  -p 3478:3478/udp \
  -p 10001:10001/udp \
  -p 8080:8080 \
  -p 8081:8081 \
  -p 8443:8443 \
  -p 8843:8843 \
  -p 8880:8880 \
  -p 6789:6789 \
  linuxserver/unifi
```

### <a name="armhf"></a> [ARMhf](#toc)
```
docker create \
  --name=unifi \
  -v <path to data>:/config \
  -e PGID=<gid> -e PUID=<uid>  \
  -p 3478:3478/udp \
  -p 10001:10001/udp \
  -p 8080:8080 \
  -p 8081:8081 \
  -p 8443:8443 \
  -p 8843:8843 \
  -p 8880:8880 \
  -p 6789:6789 \
  lsioarmhf/unifi
```

## <a name="references"></a> [References](#toc)
[^fn1]: [Dockerized UniFi Controller AMD64](https://github.com/linuxserver/docker-unifi)
[^fn2]: [Dockerized UniFi Controller armhf](https://github.com/linuxserver/docker-unifi-armhf)
