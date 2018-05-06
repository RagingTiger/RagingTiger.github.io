---
layout: post
title: "Ubiquiti UniFi Access Point Setup Guide"
date: 2018-05-04
category: [compsci, network, engineering, setup, enterprise, LAN]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Setup](#setup)
  * [UniFi Controller](#unifictrl)
* [References](#references)

<br>

## <a name="intro"></a> [Introduction](#toc)
While most of today's routers[^fn1] have a wireless access point builtin,
*Ubiquiti*[^fn2] has taken a more *modular*[^fn3] approach: all the routers are *wired* and all the *access points* are separate products. This means  when you buy a Ubiquiti router (e.g. the lovely EdgeRouter ER-X[^fn4]), you will need to either connect an existing router (that has an access point) in **bridge mode**,[^fn5] or purchase a *Ubiquiti UniFi Access Point*.[^fn6] In this guide, the focus will be on using the **UniFi AP-AC-LR**[^fn7] to serve as the wireless access to your network.

By the way, have you set up your EdgeRouter yet? If not, go ahead and follow
our guide[^fn8] to setup your router first, then return here to continue.

## <a name="setup"></a> [Setup](#toc)
**NOTE:** When first setting up your access point, *connect it directly to your EdgeRouter*. **DO NOT CONNECT TO A SWITCH!!!** A switch could interfere with the *DHCP* for the access point and make the access point unreachable or unresponsive to the UniFi Controller software.[^fn6]

### <a name="unifictrl"></a> [UniFi Controller](#toc)


## <a name="references"></a> [References](#toc)
[^fn1]: [Router Wikipedia](https://en.wikipedia.org/wiki/Router_(computing))
[^fn2]: [Ubiquiti](https://www.ubnt.com/)
[^fn3]: [Modularity](https://en.wikipedia.org/wiki/Modular_design)
[^fn4]: [EdgeRouter ER-X](https://www.ubnt.com/edgemax/edgerouter-x/)
[^fn5]: [Bridge-Mode Explained](https://en.wikipedia.org/wiki/Bridging_(networking))
[^fn6]: [UniFi AP](https://www.ubnt.com/unifi/unifi-ap/)
[^fn7]: [UniFi AP-AC-LR](https://www.ubnt.com/unifi/unifi-ap-ac-lr/)
[^fn8]: [Ubiquiti EdgeRouter Setup Guide]({{ site.baseurl }}{% link _posts/2018-04-29-ubq-erx-router-setup.md %})
