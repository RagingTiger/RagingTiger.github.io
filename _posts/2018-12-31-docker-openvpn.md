---
layout: post
title: "Damn I Love Docker: OpenVPN Server"
date: 2018-12-31
category: [docker, containers, devops, rpi, vpn]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [OpenVPN](#ovpn)
* [Installing OpenVPN](#install)
  * [AMD64](#amd)
  * [armhf](#armhf)
* [Notes On Hardware](#notes)
* [References](#references)

## <a name="install"></a> [Installing OpenVPN](#toc)

### <a name="amd"></a> [AMD64](#toc)
**NOTE**: *The following quick start install is from kylemanna's
documentation[^fn1] and uses his docker image.[^fn2]*

* Pick a name for the `$OVPN_DATA` data volume container. It's recommended to
  use the `ovpn-data-` prefix to operate seamlessly with the reference systemd
  service.  Users are encourage to replace `example` with a descriptive name of
  their choosing.

        OVPN_DATA="ovpn-data-example"

* Initialize the `$OVPN_DATA` container that will hold the configuration files
  and certificates.  The container will prompt for a passphrase to protect the
  private key used by the newly generated certificate authority.

        docker volume create --name $OVPN_DATA
        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm kylemanna/openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm -it kylemanna/openvpn ovpn_initpki

* Start OpenVPN server process

        docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn

* Generate a client certificate without a passphrase

        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm -it kylemanna/openvpn easyrsa build-client-full CLIENTNAME nopass

* Retrieve the client configuration with embedded certificates

        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm kylemanna/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

### <a name="armhf"></a> [ARMhf](#toc)
**NOTE**: *The following is the same install instructions as the AMD64, but use
an armhf image for the Raspberry Pi.*

* Pick a name for the `$OVPN_DATA` data volume container. It's recommended to
  use the `ovpn-data-` prefix to operate seamlessly with the reference systemd
  service.  Users are encourage to replace `example` with a descriptive name of
  their choosing.

        OVPN_DATA="ovpn-data-example"

* Initialize the `$OVPN_DATA` container that will hold the configuration files
  and certificates.  The container will prompt for a passphrase to protect the
  private key used by the newly generated certificate authority.

        docker volume create --name $OVPN_DATA
        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm tigerj/rpi-ovpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm -it tigerj/rpi-ovpn ovpn_initpki

* Start OpenVPN server process

        docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN tigerj/rpi-ovpn

* Generate a client certificate without a passphrase

        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm -it tigerj/rpi-ovpn easyrsa build-client-full CLIENTNAME nopass

* Retrieve the client configuration with embedded certificates

        docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm tigerj/rpi-ovpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn


## <a name="references"></a> [References](#toc)
[^fn1]: [Dockerized OVPN AMD64](https://github.com/kylemanna/docker-openvpn.git)
[^fn2]: [Dockerhub kylemanna/openvpn](https://hub.docker.com/r/tigerj/rpi-ovpn/)
