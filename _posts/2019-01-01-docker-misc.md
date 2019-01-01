---
layout: post
title: "Damn I Love Docker: Miscellaneous Docker Tips"
date: 2019-01-01
category: [docker, containers, devops, rpi, vpn]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Sync Container and Host Time](#time)
* [Container Restart Policy](#restart)
* [References](#references)

## <a name="intro"></a> [Introduction](#toc)
While there have been many posts written on this blog discussing the various
applications of **Docker**, there is need to actually discuss some of the
"finer" aspects of using docker. In this post we will introduce, and discuss
some of the "lesser known" options, commands, and features related to **Docker**.

## <a name="time"></a> [Sync Container and Host Time](#toc)
This is the best way I have found to sync the time between host and container
(for logging purposes):[^fn1]
```
-v /etc/localtime:/etc/localtime:ro
```
Simply mount the volume for your `/etc/localtime` into the container. This will
ensure that both the host machine running the **Docker Engine** and the
containers that will be run by the engine all have the same time.

## <a name="restart"></a> [Container Restart Policy](#toc)
After enough time working with docker, experimenting with different options and
commands, and even running important applications in docker, you will eventually
want to know how to "restart" a container *automatically*. This is all handled
with the `--restart` option.[^fn2]

For example, I run a *dockerized* **OpenVPN server** on my home network, and I
want to make sure that it is **ALWAYS RUNNING**. This makes sense, because I may
be in situations where I need to access the VPN, but due to a power failure at some
point, the server has been restarted. Without the `--restart` option set the
container running my VPN is not automatically restarted ... This would be a
serious pain in the **&$$**, if you catch my drift. But with the `--restart`
option, this can be alleviated like so:
```
docker create --restart=always  --name=ovpn -v /etc/localtime:/etc/localtime:ro -v $OVPN_DATA:/etc/openvpn -p 1194:1194/udp --cap-add=NET_ADMIN tigerj/rpi-ovpn
```
To clarify, the above command comes from a previous tutorial on docker
OpenVPN[^fn3], and if you look towards the beginning of the command you will see
the `--restart=always` option. As you could imagine, this ensures that your
container will (per the documentation): "Always restart the container if it
stops." Again, check out the reference[^fn2] if you would like to know more
details or other restart policies.

## <a name="references"></a> [References](#toc)
[^fn1]: [Docker / Host Time Sync](https://stackoverflow.com/a/24568137/6926917)
[^fn2]: [Start Containers Auotmatically](https://docs.docker.com/config/containers/start-containers-automatically/)
[^fn3]: [Damn I Love Docker: OpenVPN]({{site.baseurl}}/2018/12/31/docker-openvpn/)
