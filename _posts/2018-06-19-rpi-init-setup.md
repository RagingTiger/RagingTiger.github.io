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
  * [Write OS to SD Card](#writeos)
  * [Enable SSH by Default](#enablessh)
* [Initial Boot](#bootrpi)
  * [Initial Setup](#initsetup)
* [Advanced Config](#advconfig)
  * [Raspi-config](#rpiconfig)
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
$ diskutil umount /Volumes/boot
```

Let's walk through this step by step. After we
[previously flashed the SD card](#writeos), the SD card was named `boot`. When
we insert the SD card, it will be mounted on `macos` in the `/Volumes` directory
as `/Volumes/boot`. Finally, to **turn on ssh** by default, we simply need to
create a file in the root directory of the recently flashed SD card, named
`ssh`. The touch command will merely create a file, with no contents, hence
`touch /Volumes/boot/ssh`. One last thing: unmount the SD card using the
`diskutil umount` command.

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

### <a name="initsetup"></a> [Initial Setup](#toc)
Setup of the RPi is trivial using the login script we have built.[^fn5] Simply
login to the RPi, and pull down the script to be executed in *Bash* as follows:
```
$ ssh pi@raspberrypi.local
pi@raspberrypi:~ $ bash -c "$(curl -fsSL https://raw.githubusercontent.com/RagingTiger/config-rpi/master/config-rpi.sh)"
```

There are several prompts you will see: changing the default username/password,
setting up WiFi auto-join, install docker, etc. Type `Y` to the ones you want to
setup. If you choose to setup an SSH Key to do password-less entry, make sure to
pay attention to the end of the script. It explains what to do next. Here is
what it says:
```
Execute on your local machine (assumes rpi is on local network):
  $ scp ~/.ssh/id_rsa.pub USER@HOST.local:/home/USER/.ssh/authorized_keys
```
Here `USER` and `HOST` are just place holders for whatever username and hostname
, respectively, that you have chosen.

`NOTE:` Also keep in mind that this script can be run again if there is a
feature (for example to setup SSH key) that you did not select the first time,
but you decide later you would like to enable it. Just type `Y` to the prompts
for the features you want and have not already setup!

## <a name="advconfig"></a> [Advanced Config](#toc)

## <a name="references"></a> [References](#toc)
[^fn1]: [Raspberry Pi](https://www.raspberrypi.org/)
[^fn2]: [Raspibian OS](https://www.raspberrypi.org/downloads/raspbian/)
[^fn3]: [Install Docs](https://www.raspberrypi.org/documentation/installation/installing-images/)
[^fn4]: [Etcher Install](https://etcher.io/)
[^fn5]: [RagingTiger/config-rpi](https://github.com/RagingTiger/config-rpi)
[^fn6]: [Docker Install](https://docs.docker.com/install/linux/docker-ce/debian/#install-using-the-convenience-script)
[^fn7]: [Explain Shell: curl](https://explainshell.com/explain?cmd=curl+-fsSL+https%3A%2F%2Fget.docker.com+-o+get-docker.sh)
[^fn8]: [ARM Architecture](https://en.wikipedia.org/wiki/ARM_architecture)
