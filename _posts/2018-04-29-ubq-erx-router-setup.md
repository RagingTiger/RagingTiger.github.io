---
layout: post
title: "Ubiquiti EdgeRouter Setup Guide"
date: 2018-04-29
category: [compsci, network, engineering, setup, enterprise, LAN]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Setup](#setup)
  * [Initial Login](#initlogin)
* [References](#references)

<br>

## <a name="intro"></a> [Introduction](#toc)
While researching **enterprise routers** to build my own **enterprise LAN**,
I became acquainted with the Ubiquiti EdgeRouter
series.[^fn1]<sup>, </sup>[^fn2]<sup>, </sup>[^fn3] I became very impressed
with the whole **Ubiquiti** lineup, and I invested in the **EdgeRouter
ER-X**[^fn4], and the **UniFi AC LR AP**[^fn5] \(NOTE: setting up this access point will be discussed in a completely separate post[^fn6]). What follows is a brief description of how I setup my **EdgeRouter ER-x**. (NOTE: since
**macOS** is the OS that I used to setup my router, that is the OS that will
be covered in this post.)

## <a name="setup"></a> [Setup](#setup)
### <a name="initlogin"></a> [Initial Login](#initlogin)

To start with, you will need to grab an ethernet cable (e.g. RJ45 cable), and
a thunderbolt to ethernet adapter. Go ahead and connect the adapter to the
ethernet cable, and plug it into your mac. Plug the other end of the cable
into the left most ethernet port labeled **eth0** on the front of the router.
Now power up the router by plugging in the 12V power supply.

With the router powered up, the ethernet cable plugged into the port **eth0** on the front of the router, and the other end of the ethernet cable plugged into the thunderbolt adapter (which is plugged into your mac), open the **Network Preferences** under **System Preferences**.

![Network Preferences]({{site.baseurl}}/assets/img/system_preferences.png)

Then, after clicking on **Network** (as shown above), click on **Thunderbolt Ethernet** on the far left list, and select **Manually** under **Configure IPv4** and fill out the configuration as follows:

![Configure Static IP]({{site.baseurl}}/assets/img/erx_config.png)

Now *turn off the wifi* and navigate in a browser to *https://192.168.1.1*.
You may have some issues with *this connection is not secure*, ignore those.



## <a name="references"></a> [References](#toc)
[^fn1]: [EdgeRouter](https://www.ubnt.com/edgemax/edgerouter/)
[^fn2]: [EdgeMax Comparison](https://www.ubnt.com/edgemax/comparison/)
[^fn3]: [EdgeMax Products Catalog](https://www.ubnt.com/products/#edgemax)
[^fn4]: [EdgeRouter ER-X](https://www.ubnt.com/edgemax/edgerouter-x/)
[^fn5]: [UniFi AC LR AP](https://www.ubnt.com/unifi/unifi-ap-ac-lr/)
[^fn6]: [Ubiquiti UniFi AP Setup Guide]({{ site.baseurl }}{% link _posts/2018-05-04-ubq-unifi-ap-setup.md %})
