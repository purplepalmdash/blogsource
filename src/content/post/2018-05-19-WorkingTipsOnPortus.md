+++
title = "WorkingTipsOnPortus"
date = "2018-05-19T16:34:41+08:00"
description = "WorkingTipsOnPortus"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Environment
Runtime environment:    

```
OS: Ubuntu 14.04.3 LTS
docker version: 18.03.1-ce
docker-compose version: docker-compose version 1.21.2, build a133471
IP: 192.192.189.53
domain name: portus.xxxx.com
```
For installing docker:     

```
$ sudo apt-get purge lxc-docker-1.9.0
$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
$ sudo apt-get update
$ sudo apt-get install -y \
    apt-transport-https \
        ca-certificates \
            curl \
                software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
         stable"
$ sudo apt-get update
$ sudo apt-get install -y docker-ce
$ sudo apt-get install -y libyaml-dev libpython-dev
$ sudo pip uninstall docker-py
$ sudo pip uninstall docker-compose
$ sudo pip install --upgrade --force-reinstall  docker-compose
```

### Steps
Clone the source code from github:    

```
# git clone https://github.com/SUSE/Portus.git
```
Make certification in secrets folder:    

```
# cd /home/vagrant/Portus/examples/compose/secrets
# openssl req -newkey rsa:4096 -nodes -sha256 -keyout portus.key -x509 -days 3650 -out portus.crt
```
In the above steps, input following items:    

```
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Guangdong
Locality Name (eg, city) []:Guangzhou
Organization Name (eg, company) [Internet Widgits Pty Ltd]:kkkk
Organizational Unit Name (eg, section) []:cloud
Common Name (e.g. server FQDN or YOUR name) []:portus.kkkk.com
Email Address []:xxxx@xxxx.com
```

### Docker Compose File
The docker compose file is the critical for portus deployment, following is 
 my configuration file:     


```
version: "2"

services:
  portus:
    image: opensuse/portus:head
    environment:
      - PORTUS_MACHINE_FQDN_VALUE=${MACHINE_FQDN}

      # DB. The password for the database should definitely not be here. You are
      # probably better off with Docker Swarm secrets.
      - PORTUS_DB_HOST=db
      - PORTUS_DB_DATABASE=portus_production
      - PORTUS_DB_PASSWORD=${DATABASE_PASSWORD}
      - PORTUS_DB_POOL=5

      # Secrets. It can possibly be handled better with Swarm's secrets.
      - PORTUS_SECRET_KEY_BASE=${SECRET_KEY_BASE}
      - PORTUS_KEY_PATH=/certificates/portus.key
      - PORTUS_PASSWORD=${PORTUS_PASSWORD}

      # SSL
      - PORTUS_PUMA_TLS_KEY=/certificates/portus.key
      - PORTUS_PUMA_TLS_CERT=/certificates/portus.crt

      # NGinx is serving the assets instead of Puma. If you want to change this,
      # uncomment this line.
      #- RAILS_SERVE_STATIC_FILES='true'
    ports:
      - 3000:3000
    links:
      - db
    volumes:
      - ./secrets:/certificates:ro
      - static:/srv/Portus/public
    extra_hosts:
      - "portus.xxxx.com:192.192.189.53"

  background:
    image: opensuse/portus:head
    depends_on:
      - portus
      - db
    environment:
      # Theoretically not needed, but cconfig's been buggy on this...
      - CCONFIG_PREFIX=PORTUS
      - PORTUS_MACHINE_FQDN_VALUE=${MACHINE_FQDN}

      # DB. The password for the database should definitely not be here. You are
      # probably better off with Docker Swarm secrets.
      - PORTUS_DB_HOST=db
      - PORTUS_DB_DATABASE=portus_production
      - PORTUS_DB_PASSWORD=${DATABASE_PASSWORD}
      - PORTUS_DB_POOL=5

      # Secrets. It can possibly be handled better with Swarm's secrets.
      - PORTUS_SECRET_KEY_BASE=${SECRET_KEY_BASE}
      - PORTUS_KEY_PATH=/certificates/portus.key
      - PORTUS_PASSWORD=${PORTUS_PASSWORD}

      - PORTUS_BACKGROUND=true
    links:
      - db
    volumes:
      - ./secrets:/certificates:ro
    extra_hosts:
      - "portus.xxxx.com:192.192.189.53"

  db:
    image: library/mariadb:10.0.23
    command: mysqld --character-set-server=utf8 --collation-server=utf8_unicode_ci --init-connect='SET NAMES UTF8;' --innodb-flush-log-at-trx-commit=0
    environment:
      - MYSQL_DATABASE=portus_production

      # Again, the password shouldn't be handled like this.
      - MYSQL_ROOT_PASSWORD=${DATABASE_PASSWORD}
    volumes:
      - /var/lib/portus/mariadb:/var/lib/mysql
    extra_hosts:
      - "portus.xxxx.com:192.192.189.53"

  registry:
    image: library/registry:2.6
    command: ["/bin/sh", "/etc/docker/registry/init"]
    environment:
      # Authentication
      REGISTRY_AUTH_TOKEN_REALM: https://${MACHINE_FQDN}:3000/v2/token
      REGISTRY_AUTH_TOKEN_SERVICE: ${MACHINE_FQDN}:5000
      REGISTRY_AUTH_TOKEN_ISSUER: ${MACHINE_FQDN}
      #REGISTRY_AUTH_TOKEN_ISSUER: portus.test.lan
      REGISTRY_AUTH_TOKEN_ROOTCERTBUNDLE: /secrets/portus.crt

      # SSL
      REGISTRY_HTTP_TLS_CERTIFICATE: /secrets/portus.crt
      REGISTRY_HTTP_TLS_KEY: /secrets/portus.key

      # Portus endpoint
      REGISTRY_NOTIFICATIONS_ENDPOINTS: >
        - name: portus
          url: https://${MACHINE_FQDN}:3000/v2/webhooks/events
          #url: https://192.192.189.53:3000/v2/webhooks/events
          timeout: 2000ms
          threshold: 5
          backoff: 1s
    volumes:
      - /var/lib/portus/registry:/var/lib/registry
      - ./secrets:/secrets:ro
      - ./registry/config.yml:/etc/docker/registry/config.yml:ro
      - ./registry/init:/etc/docker/registry/init:ro
    ports:
      - 5000:5000
      - 5001:5001 # required to access debug service
    links:
      - portus:portus
    extra_hosts:
      - "portus.xxxx.com:192.192.189.53"

  nginx:
    image: library/nginx:alpine
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./secrets:/secrets:ro
      - static:/srv/Portus/public:ro
    ports:
      - 80:80
      - 443:443
    links:
      - registry:registry
      - portus:portus
    extra_hosts:
      - "portus.xxxx.com:192.192.189.53"

volumes:
  static:
    driver: local
```
When everything is configured, startup the service via:    

```
# docker-compose -f docker-compose.yml up
```

### Configuration
Before open your browser for accessing the portus service, do following :    

```
$ sudo echo "192.192.189.53	portus.xxxx.com">>/etc/hosts
```
Now open your browser for `https://portus.xxxx.com`:    

![/images/2018_05_19_17_36_55_626x496.jpg](/images/2018_05_19_17_36_55_626x496.jpg)

Configure the registry via:    

![/images/2018_05_19_17_37_51_488x262.jpg](/images/2018_05_19_17_37_51_488x262.jpg)

Team->Create new team, create team for kismatic deployment:    

![/images/2018_05_19_17_40_16_397x279.jpg](/images/2018_05_19_17_40_16_397x279.jpg)

Admin->User->Create new user:    

![/images/2018_05_19_17_41_59_434x313.jpg](/images/2018_05_19_17_41_59_434x313.jpg)

The created user is listed as:     

![/images/2018_05_19_17_42_41_1023x262.jpg](/images/2018_05_19_17_42_41_1023x262.jpg)

Team->members->Add members:    

![/images/2018_05_19_17_43_32_640x169.jpg](/images/2018_05_19_17_43_32_640x169.jpg)

Create a new namespace for kismatic 1.10 deployment images:    

![/images/2018_05_19_17_44_28_440x297.jpg](/images/2018_05_19_17_44_28_440x297.jpg)

You can easily view portus logs at dashboard:    

![/images/2018_05_19_17_45_14_584x314.jpg](/images/2018_05_19_17_45_14_584x314.jpg)

### Push images
upload the portus.crt to remote machine(kismatic deployment node):    

```
# scp ./portus.crt  kkkkk@192.192.189.1:/home/kkkkk/
root@registry3:~/Portus/examples/compose/secrets# pwd
/home/vagrant/Portus/examples/compose/secrets
```
Add the crt file into your system folder and trust this file, take ArchLinux
for example:     

```
$ sudo cp portus.crt /etc/ca-certificates/trust-source/anchors/portus.xxxx.com.crt
$ sudo update-ca-trust
$ sudo trust extract-compat
$ sudo systemctl restart docker
$ sudo docker login portus.xxxx.com:5000
$ sudo docker login portus.xxxx.com
Username: kismatic
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Are you sure you want to proceed? [y/N] y
Login Succeeded
```
kismatic configuration items:     

```
# vim kismatic-cluster.yaml
docker_registry:

  # IP or hostname and port for your registry.
  server: "portus.xxxx.com:5000/kismatic110"

  # Absolute path to the certificate authority that should be trusted when
  # connecting to your registry.
  CA: "/home/xxxxx/portus.xxxx.com.crt"

  # Leave blank for unauthenticated access.
  username: "kismatic"

  # Leave blank for unauthenticated access.
  password: "xxxxxxxx"
# ./kismatic seed-registry --verbose
```
Now you will see the output for uploading:    

![/images/2018_05_19_18_07_42_537x460.jpg](/images/2018_05_19_18_07_42_537x460.jpg)

namespace for kismatic110:    

![/images/2018_05_19_18_08_08_881x445.jpg](/images/2018_05_19_18_08_08_881x445.jpg)

