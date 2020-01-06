---
layout: post
title: "Damn I Love Docker: Local DNS With CoreDNS"
date: 2020-01-03
category: [docker, containers, devops, DNS]
---
### <a name="toc"></a> Table of Contents
* [Synopsis](#synop)
* [Introduction](#intro)
* [Getting Started](#setup)
  * [CoreDNS Config](#config)
  * [DNS Zone File](#zonefile)
  * [Docker Deploy](#deploy)
* [Using Your Local Name Server](#usage)
  * [Testing](#test)
  * [macOS DNS Resolver](#mac)
* [References](#references)

## <a name="synop"></a> [Synopsis](#toc)
Here we are going to discuss how you can setup a local DNS[^fn1][^fn2] server
using docker[^fn3][^fn4] and CoreDNS.[^fn5][^fn6]

## <a name="intro"></a> [Introduction](#toc)
If you are not already familiar with how DNS[^fn1][^fn2] works, then basically
it is an address book. In this address book are stored the IP addresses[^fn7] and
their corresponding `Domain Names` (i.e. google.com, youtube.com, amazon.com,
etc...). When you type `google.com` in your browser, your browser will first
attempt to resolve this `Domain Name` by querying whatever `Name Servers` your
computer has set in its local configuration (Note: this is most often set
automatically by the router on the wired/wireless network when you first join).

The result of this query is the `resolution` of the `Domain Name` to its
corresponding address (e.g. while google.com has multiple IP addresses, one of
the returned addresses is `108.177.122.113`). This can be seen below in the
following example using the well known tool `dig`:[^fn8][^fn9]
```
$ dig google.com

; <<>> DiG 9.10.6 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10808
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             76      IN      A       108.177.122.113
google.com.             76      IN      A       108.177.122.139
google.com.             76      IN      A       108.177.122.138
google.com.             76      IN      A       108.177.122.102
google.com.             76      IN      A       108.177.122.101
google.com.             76      IN      A       108.177.122.100

;; Query time: 7 msec
;; SERVER: 192.168.2.1#53(192.168.2.1)
;; WHEN: Fri Jan 03 10:09:45 EST 2020
;; MSG SIZE  rcvd: 135

```
We can see from the above example that there are several `IP addresses`
associated for `google.com` (including the `108.177.122.113` address we noted
earlier). Now your browser can actually send the desired `HTTP request`[^fn10]
to Google's servers at `108.177.122.113`, which will in turn respond with the
desired `HTML/CSS/Javascript` that comprise its search engine interface. All
`DNS` really did is allow you to **"lookup"** the address for Google's server.
In a very simplified way, this is how `DNS` works.

But this example only applies to externally located hosts (e.g. the servers
running websites like `google.com`, `youtube.com`, and `amazon.com`). What if
you wanted to create `Domain Names` and the corresponding `DNS Records` for your
local hosts/services running on your local network? This is where you would
want to have a locally running `DNS server`, with entries composed of `Domain
Names` pointing to `local IP addresses`on your *local* network.

## <a name="setup"></a> [Getting Started](#toc)
### <a name="config"></a> [CoreDNS Config](#toc)
To start with we need to create the config file for CoreDNS. This file is
called a `Corefile` and an example can be seen below
```
$ mkdir -p local-dns/root
$ cd local-dns
$ cat <<EOF > root/Corefile
.:53 {
    forward . 8.8.8.8 9.9.9.9
    log
    errors
}

example.com:53 {
    file /root/db.example
    log
    errors
}
EOF
```
First we start by using `mkdir` to create the directories to store our config
files. Then we use the `cat` command and a `heredoc` to write to the `Corefile`.
The first section of this Corefile (i.e. the `.:53`) corresponds to all DNS
queries that do no match any of the other domains listed in the config file
(in this instance we mean the `example.com` domain listed underneath). What
we are saying with this first section is *"any query that does not directly
match any other domains listed in this Corefile, forward that query to the
name servers at 8.8.8.8 and 9.9.9.9"*. These are Google's DNS servers (8.8.8.8
and 9.9.9.9 respectively). This works out well because not all of our DNS
queries are going to correspond to local domains, we still need to send out
queries for external domains out on then WAN[^fn11] (i.e. internet).

If you have not already guessed, the last section specifically matches any
DNS queries for the domain `example.com`. Here you are telling the CoreDNS
server to specifically reference the `file` located at `/root/db.example` for
every DNS query that specifies `example.com`. The domain name here can be
whatever you want and as many as you want, but you would need a separate `db.*`
file for each domain you are referencing in this `Corefile`:
```
foobarbaz.org:53 {
    file /root/db.foobarbaz
    log
    errors
}

helloworld.com:53 {
    file /root/db.helloworld
    log
    errors
}
```
In the above example we created rules to add to the `Corefile` for the domains
`foobarbaz.org` and `helloworld.com`. Notice we have uniquely named `db.*` files
for `foobarbaz.db` and `helloworld.db` respectively.

### <a name="zonefile"></a> [DNS Zone File](#toc)
Back to our original example, let us now dive into the `db.example`[^fn12] file we
referenced in the `Corefile` above, and look at how it can be setup to work:
```
$ cat <<EOF > root/db.example
$ORIGIN example.com.  ; designates the start of this zone file in the namespace
$TTL 1h               ; default expiration time of all resource records without their own TTL value
@                 IN  SOA     ns.example.com. rtiger.example.com. (
                                  2020010510     ; Serial
                                  1d             ; Refresh
                                  2h             ; Retry
                                  4w             ; Expire
                                  1h)            ; Minimum TTL
@                 IN  A       192.168.1.20       ; Local IPv4 address for example.com.
@                 IN  NS      ns.example.com.    ; Name server for example.com.
ns                IN  CNAME   @                  ; Alias for name server (points to example.com.)
webblog           IN  CNAME   @                  ; Alias for webblog.example.com
netprint          IN  CNAME   @                  ; Alias for netprint.example.com
```
We will leave an extensive explanation of the `zone file` for further
reference,[^fn12] and will instead attempt a brief explanation of the above
values.

As you can see at the top of the file is the `$ORIGIN` keyword, which merely
sets the global `ORIGIN` domain name (i.e. in this case `example.com`). The
`$ORIGIN` value can then be referenced using the `@` symbol throughout the
zone file. You can see the use of the `@` symbol to replace the `example.com`
domain name through various records. Again, please refer to the zone file
reference[^fn12] for information on the `$TTL` value.

Next we see several *resource records* (also shortened to `RR`) starting
directly below the `$TTL` value:
```
$ORIGIN example.com.  ; designates the start of this zone file in the namespace
$TTL 1h               ; default expiration time of all resource records without their own TTL value

; =============================== Resource Records ==============================

@                 IN  SOA     ns.example.com. rtiger.example.com. (
                                  2020010510     ; Serial
                                  1d             ; Refresh
                                  2h             ; Retry
                                  4w             ; Expire
                                  1h)            ; Minimum TTL
@                 IN  A       192.168.1.20       ; Local IPv4 address for example.com.
@                 IN  NS      ns.example.com.    ; Name server for example.com.
ns                IN  CNAME   @                  ; Alias for ns.example.com
webblog           IN  CNAME   @                  ; Alias for webblog.example.com
netprint          IN  CNAME   @                  ; Alias for netprint.example.com
```
More information on the `resource records` can be found in the references[^fn13]
located in the reference section. What is important to understand here is that
we have an `address record` with the label `A` that connects the domain name
`example.com` to the IP address `192.168.1.20`. This is the actual address for
`example.com`. The other resource records for `CNAME` are what are known as
`canonical name` records. Here we use some shorthand to alias the domain `ns`,
`weblog`, and `netprint` (which correspond to `ns.example.com`,
`webblog.example.com`, `netprint.example.com` respectively) to `example.com`
(again we used the `@` symbol here as a reference). This is simply because all
of the subdomain names are all pointing to the same IP address as `example.com`
so we can simply point them to the `example.com` domain. This makes management
much easier, because should we need to change the IP address associated with
`example.com`, we will not have excessive multiple entries to change.

Finally we have the `NS` and `SOA` resource records. The `NS` records simply
stand for `name server` and allow us to declare a name server (in this case
we use the `ns.example.com` domain which is actually an alias or `CNAME` for
`example.com`). The `SOA` record is the only record specifically required by
the zone file and is short for `Start of Authority`. This DNS record has a few
different features compared to previously described `A`, `CNAME`, and `NS`
records. It requires a name server address (in this case the `ns.example.com`
domain name) that hosts the `A` record for the `$ORIGIN` domain (in this case
`example.com`). In this specific example the name server is the same as the
server hosting this `zone file`, that is why the name server is this example
is merely an alias to `example.com`. It also requires an administrator contact
(in this case the `rtiger.example.com`) and various time-related data (seen
above with comments to the right). Please see the references on the resource
records[^fn13] and zone file[^fn12] for more info.

### <a name="deploy"></a> [Docker Deploy](#toc)
Finally we are ready to deploy our local DNS server. We can deploy it simply as
shown below:
```
$ docker run -d \
             --name coredns \
             -v ~/local-dns/root/:/root \
             -p 53:53/udp \
             coredns/coredns -conf /root/Corefile
```
To clarify, we are mounting the directories we created earlier
`~/local-dns/root` for CoreDNS to find the config files, so be aware that if
your path is different you will need to change this value to the location of
your Corefile/db.* files. Now we need to test it to make sure it is working

## <a name="using"></a> [Using Local DNS Server](#toc)
### <a name="test"></a> [Testing](#toc)
To begin using our server, we need to first test whether our DNS request will
work. This will involve our good friend, once again, `dig`.[^fn8][^fn9] Let us
start by querying the server using its local IP address and the `example.com`
domain:
```
$ dig @192.168.1.20 example.com

; <<>> DiG 9.10.6 <<>> @192.168.1.20 example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50252
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            3600    IN      A       192.168.1.20

;; AUTHORITY SECTION:
example.com.            3600    IN      NS      ns.example.com.

;; Query time: 41 msec
;; SERVER: 192.168.1.20#53(192.168.1.20)
;; WHEN: Sun Jan 05 18:52:10 EST 2020
;; MSG SIZE  rcvd: 106
```
We can see in the `ANSWER SECTION` of the `dig` report that the `A` record for
`example.com` points to `192.168.1.20` just like we defined in the `zone file`.
We can also see that the `AUTHORITY SECTION` reports an `NS` record with
`ns.example.com` as the name server, just like we defined in the `zone file`.
What about our other records for `webblog.example.com` and
`netprint.example.com`:
```
$ dig @192.168.1.20 webblog.example.com netprint.example.com
; <<>> DiG 9.10.6 <<>> @192.168.1.20 webblog.example.com netprint.example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37312
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;webblog.example.com.           IN      A

;; ANSWER SECTION:
webblog.example.com.    3600    IN      CNAME   example.com.
example.com.            3600    IN      A       192.168.1.20

;; AUTHORITY SECTION:
example.com.            3600    IN      NS      ns.example.com.

;; Query time: 552 msec
;; SERVER: 192.168.1.20#53(192.168.1.20)
;; WHEN: Sun Jan 05 18:56:02 EST 2020
;; MSG SIZE  rcvd: 158

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50787
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;netprint.example.com.          IN      A

;; ANSWER SECTION:
netprint.example.com.   3600    IN      CNAME   example.com.
example.com.            3600    IN      A       192.168.1.20

;; AUTHORITY SECTION:
example.com.            3600    IN      NS      ns.example.com.

;; Query time: 10 msec
;; SERVER: 192.168.1.20#53(192.168.1.20)
;; WHEN: Sun Jan 05 18:56:02 EST 2020
;; MSG SIZE  rcvd: 160

```
We see two reports generated, both containing `CNAME` records for
`webblog.example.com` and `netprint.example.com` respectively. Our local DNS
server is working!!!

### <a name="mac"></a> [macOS DNS Resolver](#toc)
In this section we actually cover how to setup your `macOS` computer to
use the *local DNS* server we have just successfully deployed and tested. We
will be using a *macOS-specific* command line tool called `networksetup`. With
this tool we will be setting the `name servers` used for a specific interface
(in this case `Wi-Fi`):
```
$ networksetup -setdnsservers Wi-Fi 192.168.1.20
```
This will immediately point all your DNS traffic to the CoreDNS server we
deployed on host `192.168.1.20`. To reset the name servers used to the default:
```
$ networksetup -setdnsservers Wi-Fi empty
```
Now we are back the the original DNS servers being used. Confirm this with
the following:
```
$ networksetup -getdnsservers Wi-Fi
There aren't any DNS Servers set on Wi-Fi.
```
The above response is the default response for macOS (i.e. the state before
we changed it to use our local DNS servers). Note: These commands were performed
in `macOS Mojave`.

## <a name="references"></a> [References](#toc)
[^fn1]: [Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System)
[^fn2]: [What is DNS](https://www.cloudflare.com/learning/dns/what-is-dns/)
[^fn3]: [Docker Wiki](https://en.wikipedia.org/wiki/Docker_(software))
[^fn4]: [Docker Docs](https://docs.docker.com/engine/docker-overview/)
[^fn5]: [CoreDNS Docs](https://coredns.io/)
[^fn6]: [CoreDNS Github](https://github.com/coredns/coredns)
[^fn7]: [IP Adresses](https://en.wikipedia.org/wiki/IP_address)
[^fn8]: [dig Wiki](https://en.wikipedia.org/wiki/Dig_(command))
[^fn9]: [dig man page](https://linux.die.net/man/1/dig)
[^fn10]: [Hypertext Transfer Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
[^fn11]: [WAN](https://en.wikipedia.org/wiki/Wide_area_network)
[^fn12]: [Zone File](https://en.wikipedia.org/wiki/Zone_file)
[^fn13]: [Resource Records](https://en.wikipedia.org/wiki/List_of_DNS_record_types)
