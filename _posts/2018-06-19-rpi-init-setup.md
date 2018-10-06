---
layout: post
title: "Raspberry Pi: Initial Setup and Configuration"
date: 2018-06-19
category: [compsci, setup, rpi]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Install OS](#rpios)
  * [Download OS](#downloados)
  * [Format SD Card](#sdcard)
  * [Write OS to SD Card](#writeos)
  * [Enable SSH by Default](#enablessh)
* [Initial Boot](#bootrpi)
  * [Initial Login](#initlogin)
* [Installing Software](#software)
  * [Docker](#docker)
* [References](#references)

<br>

## <a name="intro"></a> [Introduction](#toc)
Setting up a Raspberry Pi (RPi)[^fn1] can be very facile **IF** you follow the
steps laid out in this guide.

## <a name="rpios"></a> [Install OS](#toc)
The first thing you need to do, before anything else, is download the
**Operating System** (i.e. **OS**) and *write* it to an SD card. This guide
will be using the **Server OS** version (usually has the word *"lite"* appended
to the end of it).

### <a name="downloados"></a> [Download OS](#toc)
The OS can be
[downloaded here](https://www.raspberrypi.org/downloads/raspbian/).[^fn2]

![os download]({{site.baseurl}}/assets/img/rpi_setup/rpi_os_download1.png)

You should see a page similar to the above image (notice the *"lite"* word
appended to the file on the right). This will download the *server* edition of
the raspibian OS (i.e. no GUI). If desired, remote desktop can be setup later
(using VNC).

### <a name="writeos"></a> [Write OS to SD Card](#toc)
Per the install documentation on raspberrypi.org,[^fn3] you simply need to
download the _OS image_ ([as discussed previously](#downloados)), and download
and run the software called *Etcher*.[^fn4]

![Palpatine]({{site.baseurl}}/assets/img/gnosis/etcher.png)

As you can see from the above image, you first **click the left button** and
select the image you want to flash to the memory card (this will be the
os file you [downloaded earlier](#donwloados)). Then **click the middle button**
to select the drive you want to write to (this would be the SD card you want to
flash). Finally, **click the right button** to beginning writing the OS to the
SD card.

Once complete the card will be ejected, and you can simply remove it. But you
will be reinserting it to complete the next part.

### <a name="enablessh"></a> [Enable SSH by Default](#toc)
Before powering up your RPi, you will want to **turn on SSH** by default. To do
this, simply reinsert the SD card that you just *flashed* with the OS file, find
where it is mounted (on mac, it is in the `/Volumes` directory), and create a
file named `ssh`. For simplicity, you can follow the below command:
```
$ touch /Volumes/boot/ssh
```

Let's walk through this step by step. After we
[previously flashed the SD card](#writeos), the SD card was named `boot`. When
we insert the SD card, it will be mounted on `macos` in the `/Volumes` directory
as `/Volumes/boot`. Finally, to **turn on ssh** by default, we simply need to
create a file in the root directory of the recently flashed SD card, named
`ssh`. The touch command will merely create a file, with no contents, hence
`touch /Volumes/boot/ssh`.

Now **SSH** is turned on, and we can login to the **RPi** and begin setting it
up.

## <a name="bootrpi"></a> [Initial Boot](#toc)
Booting up is the easier part of this process. You will need:
1. *Raspberry Pi*
2. *Power Supply*
3. *Ethernet Cable*
4. *Case* (optional)

First, if you have a case, go ahead and insert the RPi into the case. Next,
insert the SD card into the RPi. Then connect the ethernet cable to
the RPi, and then into your network (i.e. switch, router, etc ...). Finally
connect the power supply to the RPi, and you now have a running RPi on your
network!

The next step is logging in, configuring the system, and optionally installing
additional software

### <a name="initlog"></a> [Initial Login](#toc)
The initial login is trivial. With the RPi booted, and connected to your
network, you should be able to simply connect to it by issuing the following:
```
$ ssh pi@raspberrypi.local
pi@raspberrypi:~ $
```

Upon logging in, go ahead and run this command to begin configuring some of the
*"annoying"* setup warnings that will **only go away** if you configure them
once and for all:
```
pi@raspberrypi:~ $ sudo raspi-config
```

After logging in initially, you will absolutely want to change the default
username and password on the RPi. To this end, there is a script[^fn5] that we can
use to setup a new user on the RPi, and remove the old *"pi"* user. While still
logged into your RPi, execute the following:
```
pi@raspberrypi:~ $ bash -c "$(curl -fsSL https://raw.githubusercontent.com/RagingTiger/create-rpi-user/master/create-user.sh)"
```

This will prompt you for a new user and password. Also, make sure to pay
attention to the end of the script. It explains what to do next to really
complete setting your new username and password. Here is what it says:
```
Execute on your local machine (assumes rpi is on local network):
  $ scp ~/.ssh/id_rsa.pub USER@raspberrypi.local:/home/USER/.ssh/authorized_keys

After keys are copied, logout of pi user and execute the following:
  $ ssh USER@raspberrypi.local

Followed by:
  USER@raspberrypi:~ $ sudo deluser pi && sudo rm -rf /home/pi
```

Here `USER` is just a place holder for whatever username you chose.

## <a name="software"></a> [Install Software](#toc)
In this section we will discuss some optional software that you may want to
consider installing on the RPi.

### <a name="docker"></a> [Docker](#toc)
Installing *Docker* on RPi[^fn6] is as simple as, logging in, running `curl`,
and executing the subsequently downloaded *shell script*:
```
$ ssh pi@raspberrypi.local
pi@raspberrypi:~ $ cd /tmp/
pi@raspberrypi:/tmp $ curl -fsSL https://get.docker.com -o get-docker.sh
pi@raspberrypi:/tmp $ less get-docker.sh
pi@raspberrypi:/tmp $ sudo sh get-docker.sh
```

Let's breakdown what these commands are doing. First we are logging into the RPi
and then changing the directory to the `/tmp` directory. Then we are executing
the curl command (with options)[^fn7] which is downloading and saving a shell
script named `get-docker.sh`. This script can then be opened for review using
the `less` command (optional, but a good practice to be into). Finally the
*get-docker.sh* shell script is being executed with root privileges using `sudo`
and `sh`.

*This will then start the shell script and install docker on your RPi!*

You are now all ready to deploy and/or run containers on your RPi, but keep in
mind one crucial caveat: containers compiled automatically on dockerhub, or
compiled on **non ARM** CPU architectures will not run on your RPi. If you try
to run them, your CPU (which has an ARM architecture),[^fn8] will issue an error
similar to this:
```
$ standard_init_linux.go:190: exec user process caused "exec format error"
```

If you see this error, you are trying to run a container, on your machine's
architecture, that was compiled for a *different architecture*.

## <a name="references"></a> [References](#toc)
[^fn1]: [Raspberry Pi](https://www.raspberrypi.org/)
[^fn2]: [Raspibian OS](https://www.raspberrypi.org/downloads/raspbian/)
[^fn3]: [Install Docs](https://www.raspberrypi.org/documentation/installation/installing-images/)
[^fn4]: [Etcher Install](https://etcher.io/)
[^fn5]: [RagingTiger/create-rpi-user](https://github.com/RagingTiger/create-rpi-user)
[^fn6]: [Docker Install](https://docs.docker.com/install/linux/docker-ce/debian/#install-using-the-convenience-script)
[^fn7]: [Explain Shell: curl](https://explainshell.com/explain?cmd=curl+-fsSL+https%3A%2F%2Fget.docker.com+-o+get-docker.sh)
[^fn8]: [ARM Architecture](https://en.wikipedia.org/wiki/ARM_architecture)
