---
layout: post
title: "Damn I Love Docker: Docker Swarm Initial Setup"
date: 2019-12-05
category: [docker, containers, devops, rpi, cluster, swarm]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Getting Started](#setup)
* [References](#references)

## <a name="intro"></a> [Introduction](#toc)
A brief walkthrough on setting up your first `docker swarm`[^fn1]

## <a name="setup"></a> [Getting Started](#toc)
It may go without saying that you need to have docker already installed[^fn2]
before proceeding, but assuming docker has been installed on *all hosts*, then
creating a swarm is trivial. First choose the initial **manager node** (this is
simply the node that you want to manage the other nodes in the cluster). If you
are working with a single node cluster (i.e. one host machine) then that will be
your **manager**.

Login to the initial manager node and execute the following:
```
$ docker swarm init

Swarm initialized: current node (y4vkoat0deioflw53cb2ie0ij) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1ce9a31d1c3b4b1ce9a31d1c3b4bdf52a1aea30359c712-5cggaqt83z5b26w17lceb7if9 192.168.100.20:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

```
Basically the output of the command is telling you that it has create the swarm
and set the current node as manager. It also tells you to run the *following
command* on the **worker nodes**:
```
$ docker swarm join --tokenSWMTKN-1-1ce9a31d1c3b4b1ce9a31d1c3b4bdf52a1aea30359c712-5cggaqt83z5b26w17lceb7if9 192.168.100.20:2377
```
Every node (i.e. host machine) on your network that you run the above command on
will *join the swarm* you just created and print out the following:
```
This node joined a swarm as a worker.
```
This lets you know that you have successfully added a new **worker node** to
the cluster. (NOTE: if you have a single node cluster this is not necessary as
the managing node is the only node in the cluster).

The last bit of information you are given in the above `docker swarm init`
command tells you how to add more *managers* to the cluster:
```
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
With this your initial cluster is complete.

## <a name="references"></a> [References](#toc)
[^fn1]: [Docker Swarm](https://docs.docker.com/engine/swarm/)
