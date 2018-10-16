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
review the guide on
[initial setup of an RPi server]({{site.baseurl}}/2018/06/19/rpi-init-setup/)),
you can actually run the dockerized UniFi controller on the Raspberry Pi, and use it as your UniFi
server!!!!

To get started, we simply need to install docker on the RPi. Again, this is
handled by the install script mentioned above,[^fn6] *BUT* if you would like
to install it directly simply use *Docker's* automated installer as follows:
```
# NOTE: we assume you are logged into the RPi
$ curl -sSL https://get.docker.com | sh
```

Once docker is installed, we can simply run the dockerized UniFi controller
(and pull down its image) as follows:
```
sudo docker run -d --restart=always --name=unifi   
-v /home/$USER/unifi_ctrl_config:/config -e PGID=1001 -e PUID=1001    
-p 3478:3478/udp -p 10001:10001/udp -p 8080:8080 -p 8081:8081 -p 8443:8443
-p 8843:8843 -p 8880:8880 -p 6789:6789 lsioarmhf/unifi
```

Let's breakdown what this is doing, so we can be more secure about running this
massively large command. First off, the `-d` option simply turns on the *daemon*
feature that allows the container to run in the background, and not *block* in
the terminal. Next the `--restart=always` simply insures that if the server
crashes, or the system reboots, the container will simply reboot as well.
With `--name=unifi`, this allows us to actually address this "specific"
container by name (instead of the long complicated *hash id* that you won't ever
remember).

The remaining options `-v, -e, -p` are semi-transparent. `-v` simply mounts
a *"volume"* into the container as another directory. Here we are mounting the
directory `unifi_ctrl_config` with path `/home/$USER/`, as the `/config`
directory in the container. This means, if you login to the container and `cd`
to the `/config` directory, you will see the contents of the
`/unifi_ctrl_config` directory. The `-e` option simply sets an environment
variable (in this case `$PGID` and `$PUID`). Finally the `-p` option sets the
port mappings for the internal network of the container, to the external network
of the *system running the container*. This means, for example, that traffic
sent on port `8080`, to the RPi will be forwarded to the port `8080` on the
container (this is akin to port forwarding on a router[^fn7]). Finally, the
`lsioarmhf/unifi` is simply the dockerhub account name, and image name (here
`lsioarmhf` is the account name and `unifi` is the image name) that we want
to pull/run.

And with that, you should have a UniFi Controller running on your RPi that
you can access from your network.

## <a name="references"></a> [References](#toc)
[^fn1]: [UniFi](https://unifi-sdn.ubnt.com/#consolidate)
[^fn2]: [UniFi Controller Download](https://www.ubnt.com/download/unifi/)
[^fn3]: [Dockerized UniFi Controller](https://github.com/linuxserver/docker-unifi)
[^fn4]: [Docker Docs](https://docs.docker.com/engine/examples/)
[^fn5]: [Raspberry Pi Wikipedia](https://en.wikipedia.org/wiki/Raspberry_Pi)
[^fn6]: [Raspberry Pi: Initial Setup and Configuration]({{site.baseurl}}/2018/06/19/rpi-init-setup/)
[^fn7]: [Ubiquiti EdgeRouter Configuration: DHCP Reservation and Port Forwarding]({{site.baseurl}}/2018/05/17/ubq-erx-config-portfowarding/)
