+++
title = "OnCustomizationOfUbuntuISO"
date = "2018-05-22T09:10:27+08:00"
description = "OnCustomizationOfUbuntuISO"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Custom Packages
Install docker and docker-compose

```
# apt-get install -y docker.io docker-compose
# systemctl enable docker
```

Pre-load docker images:   

```
# vim /bin/loaddocker.sh
if [[ $(sudo docker images | grep registry) ]]; then
    echo "there are files"
else
    docker load</usr/local/images/1.tar
    docker load</usr/local/images/2.tar
    docker load</usr/local/images/3.tar
    docker load</usr/local/images/4.tar
fi
# chmod 777 /bin/loaddocker.sh
```
Add this command to systemd:    

```
# vim /etc/systemd/system/docker-infra.service
[Unit]
Description=Docker infra
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/bin/loaddocker.sh;/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml up
ExecStop=/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml stop

[Install]
WantedBy=default.target
# systemctl enable docker-infra.service
```


