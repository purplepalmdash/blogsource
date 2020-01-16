+++
title = "tipsOnnextcloud"
date = "2020-01-16T17:46:41+08:00"
description = "Tipsonnextcloud"
keywords = ["Linux"]
categories = ["Linux"]
+++
Dockerfile:   

```
version: '3' 

services:

  proxy:
    image: jwilder/nginx-proxy:alpine
    labels:
      - "com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true"
    container_name: nextcloud-proxy
    networks:
      - nextcloud_network
    ports:
      - 0.0.0.0:80:80
      - 0.0.0.0:443:443
    volumes:
      - ./proxy/conf.d:/etc/nginx/conf.d:rw
      - ./proxy/vhost.d:/etc/nginx/vhost.d:rw
      - ./proxy/html:/usr/share/nginx/html:rw
      - ./proxy/certs:/etc/nginx/certs:ro
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/tmp/docker.sock:ro
    restart: unless-stopped
  
  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: nextcloud-letsencrypt
    depends_on:
      - proxy
    networks:
      - nextcloud_network
    volumes:
      - ./proxy/certs:/etc/nginx/certs:rw
      - ./proxy/vhost.d:/etc/nginx/vhost.d:rw
      - ./proxy/html:/usr/share/nginx/html:rw
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    restart: unless-stopped
  db:
    image: mariadb
    container_name: nextcloud-mariadb
    networks:
      - nextcloud_network
    volumes:
      - db:/var/lib/mysql
      - /etc/localtime:/etc/localtime:ro
    environment:
    # Create a root password for the maraiadb instance.
      - MYSQL_ROOT_PASSWORD=engine123
    # Create a password for the nextcloud users.  If you have to manually connect your database you would use the nextcloud user and this password.
      - MYSQL_PASSWORD=engine123
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
    restart: unless-stopped
  
  app:
    image: nextcloud:latest
    container_name: nextcloud-app
    networks:
      - nextcloud_network
    depends_on:
      - letsencrypt
      - proxy
      - db
    volumes:
      - nextcloud:/var/www/html
      - ./app/config:/var/www/html/config
      - ./app/custom_apps:/var/www/html/custom_apps
      - ./app/data:/var/www/html/data
      - ./app/themes:/var/www/html/themes
      - /etc/localtime:/etc/localtime:ro
    environment:
    # The VIRTUAL_HOST and LETSENCRYPT_HOST should use the same publically reachable domain for your nextlcloud instance.
      - VIRTUAL_HOST=mynextcloud.mooo.com
      - LETSENCRYPT_HOST=mynextcloud.mooo.com
    # This needs to be a real email as it will be used by let's encrypt for your cert and is used to warn you about renewals.
      - LETSENCRYPT_EMAIL=feipyang@gmail.com
    restart: unless-stopped
  collab:
    image: collabora/code
    container_name: nextcloud-collab
    networks:
      - nextcloud_network
    depends_on:
      - proxy
      - letsencrypt
    cap_add:
     - MKNOD
    ports:
      - 0.0.0.0:9980:9980
    environment:
    # This nees to be the same as what you set your app domain too (ex: cloud.domain.tld).
    #- domain=cloud\\.DOMAIN\\.TDL
      - domain=mynextcloud\.mooo\.com
      - username=admin
    # Create a passoword for the collabora office admin page.
    #- password=CREATE-A-SECURE-PASSWORD-HERE
      - password=engine123
      - VIRTUAL_PORT=9980
      - extra_params=--o:ssl.enable=false --o:ssl.termination=true
#      - VIRTUAL_PROTO=https
#      - VIRTUAL_PORT=443
    # The VIRTUAL_HOST and LETSENCRYPT_HOST should use the same publically reachable domain for your collabora instance (ex: office.domain.tld).
      - VIRTUAL_HOST=myoffice.mooo.com
      - LETSENCRYPT_HOST=myoffice.mooo.com
    # This needs to be a real email as it will be used by let's encrypt for your cert and is used to warn you about renewals.
      - LETSENCRYPT_EMAIL=feipyang@gmail.com
    restart: unless-stopped 
volumes:
  nextcloud:
  db: 
  
networks:
  nextcloud_network:

```

For installing apps:    

```
sudo docker cp Client.php fc18391f0a0a:/var/www/html/lib/private/Http/Client/Client.php
Timeout from 30 to 300, then you could install apps. 
```
