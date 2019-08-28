+++
title = "DockerizeES"
date = "2019-08-27T09:09:14+08:00"
description = "DockerizeES"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
The `docker-compose.yml` is listed as following:    

```
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.1.3
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - node.name=coreos-1
      - node.master=true
      - node.data=true
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
      - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 192.168.122.31:9200:9200
      - 192.168.122.31:9300:9300

volumes:
  esdata1:
    driver: local
```
The `elasticsearch.yml` file is listed as following:    

```
network.host: 0.0.0.0
discovery.zen.ping.unicast.hosts: ["192.168.122.31"]
network.publish_host: 192.168.122.31
```
Thus you could use docker-compose for setting the cluster.    

### tips
master and worker:    

```
for master node, node.master=true
for worker node, node.master=false
```


