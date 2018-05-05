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
  * [Basic Config](#wizard)
  * [Advance Config](#advconfig)
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

## <a name="setup"></a> [Setup](#toc)
### <a name="initlogin"></a> [Initial Login](#toc)

To start with, you will need to grab an ethernet cable (e.g. RJ45 cable), and
a thunderbolt to ethernet adapter. Go ahead and connect the adapter to the
ethernet cable, and plug it into your mac. Plug the other end of the cable
into the left most ethernet port labeled **eth0** on the front of the router.
Now power up the router by plugging in the 12V power supply.

With the router powered up, the ethernet cable plugged into the port **eth0** on the front of the router, and the other end of the ethernet cable plugged into the thunderbolt adapter (which is plugged into your mac), open the **Network Preferences** under **System Preferences**.

![Network Preferences]({{site.baseurl}}/assets/img/ers/system_preferences.png)

Then, after clicking on **Network** (as shown above), click on **Thunderbolt Ethernet** on the far left list, and select **Manually** under **Configure IPv4** and fill out the configuration as follows:

![Configure Static IP]({{site.baseurl}}/assets/img/ers/erx_config.png)

Now *turn off the wifi* and navigate in a browser to *https://192.168.1.1*.
You may have some issues with *this connection is not secure*, ignore them and proceed to the website (see below for an example on chrome):

![Insecure Warning]({{site.baseurl}}/assets/img/ers/insecure_warning1.png)

![Proceed to Site]({{site.baseurl}}/assets/img/ers/insecure_warning2.png)

Once you have correctly *navigated to the website*, you should see a **login page** like the following:

![Login Page]({{site.baseurl}}/assets/img/ers/loginpage1.png)

Enter the **login credentials** as follows:
```
Username: ubnt
Password: ubnt
```

![Login Credentials]({{site.baseurl}}/assets/img/ers/loginpage2.png)

These are the *default credentials* and you will want to change them (this will be covered later). If successful, you should see a *dashboard* for the *EdgeOS*:

![EdgeOS Dashboard]({{site.baseurl}}/assets/img/ers/dashboard1.png)

You have completed the *initial login* to the router! Now in the **next section** we will look at how to setup the *basic configuration* (i.e. get the router working with internet access and all your hosts on the same LAN). From this basic configuration we will look at how to alter the configurations for more advanced setups (e.g. multiple LANs).

### <a name="wizard"></a> [Basic Configuration](#toc)
To begin the basic configuration, we need to click on the **Wizards** tab in the upper right portion of the dashboard (just below the **Toolbox** button):

![EdgeOS Wizards]({{site.baseurl}}/assets/img/ers/dashboard2.png)

Once you are on the **Wizards page**, find the *list of wizards on the left side*, and click on **WAN + 2LAN2**:

![WAN+2LAN2]({{site.baseurl}}/assets/img/ers/baseconfig1.png)

Just leave the configuration setup as is, do not change anything, and click **apply** to implement the configuration:

![Apply Config]({{site.baseurl}}/assets/img/ers/baseconfig2.png)

After you *click apply* you will see a series of prompts asking you about *applying changes*, and then *rebooting*, and then *are you sure*. **Just click yes**:

![Reboot1]({{site.baseurl}}/assets/img/ers/restart1.png)
![Reboot2]({{site.baseurl}}/assets/img/ers/restart2.png)
![Reboot3]({{site.baseurl}}/assets/img/ers/restart3.png)
![Reboot4]({{site.baseurl}}/assets/img/ers/restart4.png)

Now, the router will be rebooting, and loading the newly setup basic configuration. While it is rebooting, disconnect the *ethernet cable* from the router port **eth0** and connect it to **eth1**. Open up the network preferences, find **Thunderbolt Ethernet** in the list on the left of the page, click on **Configure IPv4**, and switch it to **Using DHCP** like the following:

![DHCP Config]({{site.baseurl}}/assets/img/ers/dhcp_config1.png)

Once the *router finishes booting up*, it will assign you an *address* (here it was 192.168.1.38), and you will be on the network. The only thing left now is to plug in the *ethernet from your modem* into the router ethernet port **eth0**. *This completes the basic configuration*. (woooooooooh!!) Your network should now be working, and all the hosts connected to ports **eth1-4** will be able to see and access each other. This is the typical router setup. From here there are some options: 1. continue with the network setup by *configuring a wireless access point* (see our guide[^fn6]), 2. begin *configuring a more advanced setup*. The latter will be covered in the next section

### <a name="advconfig"></a> [Advanced Configurations](#toc)


## <a name="references"></a> [References](#toc)
[^fn1]: [EdgeRouter](https://www.ubnt.com/edgemax/edgerouter/)
[^fn2]: [EdgeMax Comparison](https://www.ubnt.com/edgemax/comparison/)
[^fn3]: [EdgeMax Products Catalog](https://www.ubnt.com/products/#edgemax)
[^fn4]: [EdgeRouter ER-X](https://www.ubnt.com/edgemax/edgerouter-x/)
[^fn5]: [UniFi AC LR AP](https://www.ubnt.com/unifi/unifi-ap-ac-lr/)
[^fn6]: [Ubiquiti UniFi AP Setup Guide]({{ site.baseurl }}{% link _posts/2018-05-04-ubq-unifi-ap-setup.md %})
