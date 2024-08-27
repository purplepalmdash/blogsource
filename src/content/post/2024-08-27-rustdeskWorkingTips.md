+++
title= "rustdeskWorkingTips"
date = "2024-08-27T14:05:28+08:00"
description = "rustdeskWorkingTips"
keywords = ["Technology"]
categories = ["Technology"]
+++
Server startup via:    

```
# vim docker-compose.yml
version: '3'

networks:
  rustdesk-net:
    external: false

services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    image: rustdesk/rustdesk-server:latest
    command: hbbs -r 127.0.0.1:21117
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: rustdesk/rustdesk-server:latest
    command: hbbr
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    restart: unless-stopped
# docker-compose -f docker-compose.yml up -d
```
Inspect the running docker instance:    

```
 sudo docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED        STATUS       PORTS                                                                                                                                                                 NAMES
5e5ea15264d5   rustdesk/rustdesk-server:latest   "hbbs -r 127.0.0.1:2…"   2 hours ago    Up 2 hours   0.0.0.0:21115-21116->21115-21116/tcp, :::21115-21116->21115-21116/tcp, 0.0.0.0:21118->21118/tcp, :::21118->21118/tcp, 0.0.0.0:21116->21116/udp, :::21116->21116/udp   hbbs
6206c4cbb810   rustdesk/rustdesk-server:latest   "hbbr"                   2 hours ago    Up 2 hours   0.0.0.0:21117->21117/tcp, :::21117->21117/tcp, 0.0.0.0:21119->21119/tcp, :::21119->21119/tcp                                                                          hbbr
```

Configuration on this server:    

![/images/20240827_140736_x.jpg](/images/20240827_140736_x.jpg)

The key is filled with following steps:     

```
[dash@shidaarch ~]$ cd rustdeck/data/
[dash@shidaarch data]$ ls
db_v2.sqlite3  db_v2.sqlite3-shm  db_v2.sqlite3-wal  id_ed25519  id_ed25519.pub
[dash@shidaarch data]$ cat id_ed25519.pub 
8+p6ycEu7aPcLDSkzBg4Lgml3m5EbuTzzl9yRhfixCE=
```
