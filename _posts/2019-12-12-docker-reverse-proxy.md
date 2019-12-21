---
layout: post
title: "Damn I Love Docker: NGINX Reverse Proxy"
date: 2019-12-12
category: [docker, containers, devops, reverse proxy, nginx, virtual hosting]
---
### <a name="toc"></a> Table of Contents
* [Synopsis](#synop)
* [Introduction](#intro)
* [Getting Started](#setup)
  * [Docker Single Host](#sing)
    * [Configure](#config)
    * [Deploy](#deploy)
      * [Simple](#simp)
      * [Advanced](#adv)
        * [Creating Docker Network](#ntwrk)
        * [Connecting WebApps](#ntwrk-webapp)
        * [Connecting Reverse Proxy](#ntwrk-proxy)
* [References](#references)

## <a name="synop"></a> [Synopsis](#toc)
In this  tutorial we cover how to setup a reverse proxy[^fn1] using NGINX[^fn2]
running in a docker container. The emphasis in this tutorial will be primarily
on the use of a dockerized NGINX server running as a reverse proxy to allow
for the hosting of multiple websites/web apps behind a single IP address.


## <a name="intro"></a> [Introduction](#toc)
Running multiple websites/web applications from the same physical IP address
(i.e. from your home network) is a common issue many face when deploying their
websites/web apps for the first time. Basically you need something that will sit
in the front of your websites/web application servers and route traffic to them
based on the `Host` field of the request.

This is sometimes referred to as an `edge router` and is exactly the kind of
task that NGINX is suited for (Note: see Traefik[^fn3] for a more advanced/specific
reverse proxy for edge routing). While tools like Traefik[^fn3] have been
developed more recently and more specifically for this problem, NGINX is a more
`minimal viable product` and possibly more facile to deal with and setup for
your initial reverse proxy needs.

## <a name="setup"></a> [Getting Started](#toc)
Below we cover the different scenarios for setting up a reverse proxy on a
docker host.

### <a name="sing"></a> [Docker Single Host](#toc)
To clarify, we will be showing how to setup a reverse proxy using docker on a
single host **NOT** in `swarm mode`.[^fn4] The setup for a single host docker
environment will not distribute appropriately on a `docker swarm` because we
are using volumes.

A good example of how to set this up can be found in the repository:
http://github.com/RagingTiger/request-router. Here is the gist of the info
shared:

#### <a name="config"></a> [Configure](#config)
Begin by creating a directory to store your NGINX config files (this can be
anywhere you like that is accessible from the file system)
```
$ mkdir -p ~/reverse-proxy/config ~/reverse-proxy/conf.d
```
These two directories `config` and `conf.d` are where we will be storing the
config file for both the `NGINX` server and also the `domain routing` for your
backend services that sit behind the reverse proxy.

Next add the config file:
```
$ cd ~/reverse-proxy/
$ cat <<EOF > config/nginx.conf
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  ' -  [] "" '
                      '0  "" '
                      '"" ""';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
EOF
```
Here we use the `cat` command and a `heredoc`[^fn5] to create the `nginx.conf`
file in the `reverse-proxy/config/` directory. This completes the configuration
of the `NGINX` server. Now we will add some example `proxy config` files to the
`conf.d` directory.

The proxy config example given from the github repository[^fn6] simply shows
how you can write a configuration for sending traffic with a specific `host`
name or `domain name` to a specific internal address:
```
$ cat <<EOF > conf.d/example.conf
server {
    listen       80;
    server_name  example.com;

    location / {
        proxy_pass http://192.168.1.20:8080;
    }
}
EOF
```
There are a few points worth mentioning about this config file. The `listen`
keyword simply tells NGINX that any traffic it receives on `port 80`, it should
check to see if the `host` name or `domain name` matches the `server_name`[^fn7]
given in this configuration block. In this example, any request that this NGINX
server would receive on `port 80` (the default HTTP port) must be parsed to see
if the `host`/`domain name` matches the `server_name` **example.com**. If a
match is found the NGINX server will route that request to the URL and port
for `proxy_pass`[^fn8](seen above in the `location /` sub-block of the `server`
block).[^fn9] Hence traffic for **example.com** will be sent to the local
address `http://192.168.1.20` on `port 8080`.

#### <a name="simp"></a> [Deploy: Simple](#toc)
The simplest setup is to deploy a container built with a stable version of
NGINX (nginx:1.15.8-alpine at the time of this publication):
```
# run docker container in daemon mode (i.e. -d option)
$ docker run -d \
             --name=reverse-proxy \
             -v ~/reverse-proxy/config/nginx.conf:/etc/nginx/nginx.conf \
             -v ~/reverse-proxy/conf.d/:/etc/nginx/conf.d/ \
             -p 80:80 \
             nginx:1.15.8-alpine
```
Now simply add a `proxy config` file to the `reverse-proxy/conf.d/` directory
with a unique name (e.g. `andrewsblog.conf` or `webdevsite.conf`) as follows:
```
$ cat <<EOF > conf.d/webblog.conf
server {
    listen       80;
    server_name  thelifeofsarah.com;

    location / {
        proxy_pass http://192.168.1.25:5000;
    }
}
$ docker restart reverse-proxy
```
Notice after the creation of the `webblog.conf` config file we restart the
NGINX docker container using `docker restart ...`. This simply restarts NGINX
so that it will read the new configuration file. Now you should be able to send
traffic to the reverse proxy and if it gets requests with `thelifeofsarah.com`
in the `Host` header[^fn10] it will route them to `http://192.168.25:5000`.

#### <a name="adv"></a> [Deploy: Advanced](#toc)
This version of the deploy follows the same steps above, except it specifically
addresses situations where you want to run your `reverse proxy` on the same
physical docker host as your websites/web applications. This allows us to do
some work with `docker network` commands.[^fn11]

But why work with `docker network` commands? Well there are a few benefits worth
pointing out:
  + DNS using container names (e.g. --name webapp)
  + Network isolation

##### <a name="ntwrk"></a> [Creating Docker Network](#toc)
Let us start by creating a docker network named `proxy`:
```
$ docker network create proxy
```
This will create a network called `proxy` using the default driver `bridge`. See
docker network documentation[^fn12] for more information.

##### <a name="ntwrk-website"></a> [Connecting Web Apps](#toc)
Now we can add our website container to the network using a docker image called
`foobar/blog` (NOTE: this is not a real image):
```
$ docker run -d \
             --name=blog \
             --network proxy \
             foobar/blog
```
Imagine this `foobar/blog` is the image for your website/web app. Then if you
look at the options given, we passed the `proxy` network to the `docker run`
command as the `--network` option. This will connect the `blog` container to
the `proxy` network.

Pay attention to the name we gave the container, `blog`, as this name can now be
used to directly send traffic to the `blog` container by name from any other
container on the `proxy` network.[^fn12] This will come in handy in the next
section.

##### <a name="ntwrk-proxy"></a> [Connecting Reverse Proxy](#toc)
Remember the configuration file we wrote in the earlier [Deploy: Simple](#simp)
section? We are going to rewrite it here utilizing the inherent Docker
DNS[^fn12] feature.
```
$ cd ~/reverse-proxy/
$ cat <<EOF > conf.d/webblog.conf
server {
    listen       80;
    server_name  thelifeofsarah.com;

    location / {
        proxy_pass http://blog:5000; ## <--- Now using container name as IP/DN
    }
}
```
So basically docker allows you to use the name of a container (which you can
specify with the `--name` option) to send traffic to that container. In this
case, we need to configure NGINX to send traffic to our `blog` container, and we
can use the name of that container (i.e. `blog`) as the local domain name for
that container on the `proxy` network.

Next we are going to start our reverse proxy passing this `proxy` network as a
command line argument using the `--network` option:
```
$ docker run -d \
             --name=reverse-proxy \
             -v ~/reverse-proxy/config/nginx.conf:/etc/nginx/nginx.conf \
             -v ~/reverse-proxy/conf.d/:/etc/nginx/conf.d/ \
             -p 80:80 \
             --network proxy \
             nginx:1.15.8-alpine
```
Note: If you already have the a container from the previous section running
with the name `reverse-proxy` simply stop the container and remove it as such:
```
$ docker stop reverse-proxy && docker rm reverse-proxy
```

Now everything should be networked together. No one can access your `blog`
directly. They can only send traffic to the `reverse-proxy` container running
on port `80`. This reverse proxy then forwards that traffic to its internal
network (named `proxy`) that is shared by both the `blog` and `reverse-proxy`
containers.

## <a name="references"></a> [References](#toc)
[^fn1]: [Reverse Proxy](https://en.wikipedia.org/wiki/Reverse_proxy)
[^fn2]: [NGINX Wiki](https://en.wikipedia.org/wiki/Nginx)
[^fn3]: [Traefik](https://containo.us/traefik/)
[^fn4]: [Docker Swarm](https://docs.docker.com/engine/swarm/)
[^fn5]: [heredoc](https://en.wikipedia.org/wiki/Here_document)
[^fn6]: [request-router](http://github.com/RagingTiger/request-router)
[^fn7]: [NGINX server_name](http://nginx.org/en/docs/http/server_names.html)
[^fn8]: [NGINX proxy_pass](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
[^fn9]: [NGINX Request Process](http://nginx.org/en/docs/http/request_processing.html)
[^fn10]: [HTTP Request Headers](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields)
[^fn11]: [Docker Network](https://docs.docker.com/network/)
[^fn12]: [Docker DNS](https://docs.docker.com/v17.09/engine/userguide/networking/configure-dns/)
