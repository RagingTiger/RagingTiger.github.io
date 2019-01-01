---
layout: post
title: "Damn I Love Docker: Miscellaneous Docker Tips"
date: 2019-01-01
category: [docker, containers, devops, rpi, vpn]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Sync Container and Host Time](#time)
* [References](#references)

## <a name="time"></a> [Sync Container and Host Time](#toc)
This is the best way I have found to sync the time between host and container
(for logging purposes):[^fn1]
```
-v /etc/localtime:/etc/localtime:ro
```
Simply mount the volume for your `/etc/localtime` into the container.


## <a name="references"></a> [References](#toc)
[^fn1]: [Docker / Host Time Sync](https://stackoverflow.com/a/24568137/6926917)
[^fn2]: [Dockerhub kylemanna/openvpn](https://hub.docker.com/r/tigerj/rpi-ovpn/)
