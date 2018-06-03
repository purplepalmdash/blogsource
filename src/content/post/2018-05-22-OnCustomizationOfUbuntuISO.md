+++
title = "OnCustomizationOfUbuntuISO"
date = "2018-05-22T09:10:27+08:00"
description = "OnCustomizationOfUbuntuISO"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Install cubic
Install cubic via:    

```
# apt-add-repository ppa:cubic-wizard/release
# apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6494C6D6997C215E
# apt update && apt install cubic
```

### Custom Packages
Install docker and docker-compose

```
# apt-get install -y docker.io docker-compose openssh-server
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
Add dockerload service

```
# vim /etc/systemd/system/dockerload.service 
[Unit]
Description=Docker load
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/bin/loaddocker.sh
ExecStop=/usr/bin/echo hello

[Install]
WantedBy=multi-user.target

```


Add this command to systemd:    

```
# vim /etc/systemd/system/docker-infra.service
[Unit]
Description=Docker infra
Requires=docker.service
After=dockerload.service

[Service]
WorkingDirectory=/usr/local/compose/
Type=idle
Restart=always
# Remove old container items
ExecStartPre=/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml down
# Compose up
ExecStart=/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml up
# Compose stop
ExecStop=/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml stop

[Install]
WantedBy=multi-user.target

# systemctl enable docker-infra.service
```
Change the default interface to eth0/eth1, etc.    

```
# vim /etc/default/grub
net.ifnames=0 biosdevname=0
# update-grub
```

### Remove unnecessary packages
Following:    

```
Amazon
Libreoffice
Mahjongg
Mines
Shotwell
Sudoku
totem
totem-common
vino
transmission-gtk
transmission-common
remmina
eog

```
Disable ufw:    

```
# ufw disable
```

Remove command:    

```
# apt-get purge   aisleriot eog gnome-mahjongg gnome-mines gnome-sudoku hplip libreoffice-avmedia-backend-gstreamer
  libreoffice-base-core libreoffice-calc libreoffice-common libreoffice-core libreoffice-draw
  libreoffice-gnome libreoffice-gtk3 libreoffice-impress libreoffice-math libreoffice-ogltrans
  libreoffice-pdfimport libreoffice-style-breeze libreoffice-style-galaxy libreoffice-style-tango
  libreoffice-writer  python3-uno remmina remmina-plugin-rdp
  remmina-plugin-secret remmina-plugin-vnc thunderbird thunderbird-gnome-support totem totem-common
  totem-plugins transmission-common transmission-gtk  vino

```

### tips for centos

```
if [[ $(sudo docker images | grep registry) ]]; then
    echo "there are files"
else
    docker load</usr/local/images/nginx.tar.bz2
    docker load</usr/local/images/1.tar
    docker load</usr/local/images/2.tar
    docker load</usr/local/images/3.tar
    docker load</usr/local/images/4.tar
    docker run --name docker-nginx -p 8888:80 -d -v /usr/local/kismaticpkgs:/usr/share/nginx/html jrelva/nginx-autoindex
fi

```
Add service:    

```
# vim /etc/systemd/system/mynginx.service 
[Unit]
Description=mynginx
Requires=docker.service
After=docker-infra.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a docker-nginx
ExecStop=/usr/bin/docker stop -t 2 docker-nginx

[Install]
WantedBy=multi-user.target

```

