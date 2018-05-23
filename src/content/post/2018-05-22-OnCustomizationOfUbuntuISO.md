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
Change the default interface to eth0/eth1, etc.    

```
# vim /etc/default/grub
net.ifnames=0 biosdevname=0
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
