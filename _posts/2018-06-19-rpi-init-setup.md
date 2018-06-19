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
* [References](#references)

<br>

## <a name="intro"></a> [Introduction](#toc)
Setting up a Raspberry Pi[^fn1] can be very facile **IF** you follow the steps laid
out in this guide.

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
appended to the file on the right).

### <a name="sdcard"></a> [Format SD Card](#toc)
With the OS file downloaded, you now simply need to format your SD card. This
guide uses a **64Gb SD card**, and these cards require the **exFAT** format.

### <a name="writeos"></a> [Write OS to SD Card](#toc)

### <a name="enablessh"></a> [Enable SSH by Default](#toc)

## <a name="bootrpi"></a> [Initial Boot](#toc)

### <a name="initlog"></a> [Initial Login](#toc)
Upon logging in, go ahead and run this command to begin configuring some of the
*"annoying"* setup warnings that will **only go away** if you configure them
once and for all:
```
$ sudo raspi-config
```

After logging in initially, you will absolutely want to change the default
username and password on the RPi. To this end, there is a script[^fn3] that we can
use to setup a new user on the RPi, and remove the old *"pi"* user. While still
logged into your RPi, execute the following:
```
$ bash -c "$(curl -fsSL https://raw.githubusercontent.com/RagingTiger/create-rpi-user/master/create-user.sh)"
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
  $ sudo deluser pi && sudo rm -rf /home/pi
```

Here `USER` is just a place holder for whatever username you chose.

## <a name="references"></a> [References](#toc)
[^fn1]: [Raspberry Pi](https://www.raspberrypi.org/)
[^fn2]: [Raspibian OS](https://www.raspberrypi.org/downloads/raspbian/)
[^fn3]: [RagingTiger/create-rpi-user](https://github.com/RagingTiger/create-rpi-user)
