---
layout: post
title: "Ubiquiti UniFi Controller Setup Guide"
date: 2018-05-04
category: [compsci, network, engineering, setup, enterprise, LAN]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Setup](#setup)
  * [Install Locally](#local)
  * [Dockerized UniFi](#dockerunifi)
  * [Raspberry Pi UniFi](#unifipi)
* [References](#references)

<br>

## <a name="intro"></a> [Introduction](#toc)
One of the pros/cons of the *Ubiquiti UniFi*[^fn1] ecosystem is the necessity of
the *UniFi controller* software.[^fn2] While this software ultimately works
well, it does require downloading and installing.[^fn2] This can be a bit
undesirable to your veteran developer/engineer/hacker that does not want more
applications cluttering their file system.

Luckily there is a **dockerized**[^fn3] version of the application that we will
be discussing here, as well as the traditional install method, and a strategy
for deploying the **docker container**[^fn4] to a **Raspberry Pi**[^fn5] server.


## <a name="setup"></a> [Setup](#toc)

### <a name="local"></a> [Install Locally](#toc)

### <a name="dockerunifi"></a> [Dockerized UniFi](#toc)

### <a name="unifirpi"></a> [Raspberry Pi UniFi](#toc)
Assuming you have already setup a Raspberry Pi (RPi) server (if not please
review the guide on [initial setup of an RPi server]()), you can actually run
the dockerized UniFi controller on the Raspberry Pi, and use it as your UniFi
server!!!!

To get started, we simply need to install docker on the RPi

Once docker is installed, we need to make sure we have the `git` command
installed as well

Using `git clone` we will *"clone"* the source code for the
**linuxserver/unifi** docker image to the **RPi**. The default image is compiled
for a different **CPU architecture** than the **ARM architecture** that the RPi
uses. Hence, we will need the **Dockerfile** found in the **GitHub** repo to
build the docker image on the RPi (and hence have the right architecture).



## <a name="references"></a> [References](#toc)
[^fn1]: [UniFi](https://unifi-sdn.ubnt.com/#consolidate)
[^fn2]: [UniFi Controller Download](https://www.ubnt.com/download/unifi/)
[^fn3]: [Dockerized UniFi Controller](https://github.com/linuxserver/docker-unifi)
[^fn4]: [Docker Docs](https://docs.docker.com/engine/examples/)
[^fn5]: [Raspberry Pi Wikipedia](https://en.wikipedia.org/wiki/Raspberry_Pi)
