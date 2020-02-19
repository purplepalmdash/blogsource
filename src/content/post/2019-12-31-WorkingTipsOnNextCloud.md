+++
title = "WorkingTipsOnNextCloud"
date = "2019-12-31T15:50:19+08:00"
description = "WorkingTipsOnNextCloud"
keywords = ["Linux"]
categories = ["Technology"]
+++
### docker-compose file
File content:    

```
version: '2'
services: 
  nextcloud_db:
    image: mariadb
    container_name: nextcloud_db
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=mysql12345678
      - MYSQL_PASSWORD=mysql12345678
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
    restart: always

  nextcloud:
    image: nextcloud
    container_name: nextcloud_web
    ports:
      - "10388:80"
    environment:
      - UID=1000
      - GID=1000
      - UPLOAD_MAX_SIZE=5G
      - APC_SHM_SIZE=128M
      - OPCACHE_MEM_SIZE=128
      - CRON_PERIOD=15m
      - TZ=Aisa/Shanghai
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASSWORD=xxxxxxxxxxxxxxxx
      - NEXTCLOUD_TRUSTED_DOMAINS="*"
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=mysql12345678
      - MYSQL_HOST=nextcloud_db
    volumes:
      - ./nextcloud:/var/www/html
    restart: always
```
### systemd file
`/etc/systemd/system/nextcloud.service`     

```
[Unit]
Description=nextcloud
Requires=docker.service
After=docker.service

[Service]
WorkingDirectory=/media/sdb/nextcloud
Type=idle
Restart=always
# Remove old container items
ExecStartPre=/usr/bin/docker-compose -f /media/sdb/nextcloud/docker-compose.yml down
# Compose up
ExecStart=/usr/bin/docker-compose -f /media/sdb/nextcloud/docker-compose.yml up
# Compose stop
ExecStop=/usr/bin/docker-compose -f /media/sdb/nextcloud/docker-compose.yml stop

[Install]
WantedBy=multi-user.target
```
Enable and start the service thus you got the nextcloud server at `http://YourIP:10388/`, login with admin/xxxxxxxx

### Upload and update
upload the files into correspondding location and update the cache:    

```
# chmod 777 -R /media/sdb/nextcloud/nextcloud/data/xxxxx/files/xxxxx_Static/
# docker exec -u www-data nextcloud_web php occ files:scan --all
```
by now your nextcloud will update properly.   
