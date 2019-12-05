---
layout: post
title: "Damn I Love Docker: Docker Swarm Metrics"
date: 2019-12-02
category: [docker, containers, devops, rpi]
---
### <a name="toc"></a> Table of Contents
* [Introduction](#intro)
* [Deploying Metrics Stack](#metrics)
* [Using Prometheus/Granfa](#usign)
* [References](#references)

## <a name="intro"></a> [Introduction](#toc)
Here we discuss one way to monitor the nodes in a docker swarm[^fn1] cluster
using Prometheus,[^fn2] Grafana,[^fn3] and node-exporter.[^fn4]

## <a name="metrics"></a> [Metrics Stack](#toc)
NOTE: This guide assumes you already have a `docker swarm`[^fn1] setup.

### <a name="services"></a> [Node-Exporter Service](#toc)
The following command will **create** a `docker service` on your docker swarm
composed of the node-exporter application written by Prometheus.[^fn2]
```
docker service create --mode global \
                      --name=metrics_node-exporter \
                      --network=host \
                      prom/node-exporter \
                      --path.rootfs=/host
```

### <a name="stack"></a> [Prometheus/Grafana Stack](#toc)
The following command will **deploy** a `docker stack`[^fn6] composed of two
docker services[^fn5]: Prometheus[^fn2], and Grafana.[^fn3] Before we look at
how to create the docker stack we need to look at the config file for
Prometheus:
```
$ cat prometheus_grafana/prometheus.yml
global:
  scrape_interval: 1m
  scrape_timeout: 10s
  evaluation_interval: 1m

scrape_configs:
  - job_name: node-exporter
    static_configs:
      - targets:
        - 192.168.100.30:9100
        - 192.168.100.31:9100
        - 192.168.100.32:9100
```
Next we will write a **stack file** which has the same syntax as a
**docker-compose** file[^fn7]:
```
$ cat prometheus_grafana/metrics-stack.yml

version: "3.7"

volumes:
  prometheus-data:
  grafana-data:

configs:
  prometheus-config:
    file: ./prometheus.yaml

services:
  prometheus:
    deploy:
      placement:
        constraints:
          - node.role == manager
    hostname: prometheus
    configs:
      - source: prometheus-config
        target: /prometheus.yaml
    volumes:
      - prometheus-data:/prometheus
    ports:
      - 9090:9090
    image: prom/prometheus
    command: [
      --config.file, /prometheus.yaml,
      --storage.tsdb.path, /prometheus
    ]

  grafana:
    deploy:
      placement:
        constraints:
          - node.role == manager
    hostname: grafana
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - 3000:3000
    image: grafana/grafana

```
Finally we can deploy the stack as follows:
```
$ docker stack deploy -c prometheus_grafana/metrics_stack.yml metrics
```

## <a name="references"></a> [References](#toc)
[^fn1]: [Docker Swarm](https://docs.docker.com/engine/swarm/)
[^fn2]: [Prometheus](https://prometheus.io/)
[^fn3]: [Grafana](https://grafana.com/)
[^fn4]: [Node-Exporter](https://prometheus.io/docs/guides/node-exporter/)
[^fn5]: [Docker Service](https://docs.docker.com/engine/reference/commandline/service/)
[^fn6]: [Docker Stack](https://docs.docker.com/engine/reference/commandline/stack/)
[^fn7]: [Docker Stack File Syntax](https://docs.docker.com/engine/reference/commandline/stack_deploy/#compose-file)
