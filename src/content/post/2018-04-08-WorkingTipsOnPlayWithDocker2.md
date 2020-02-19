+++
title = "WorkingTipsOnPlayWithDocker2"
date = "2018-04-08T15:29:33+08:00"
description = "WorkingTipsOnPlayWithDocker2"
keywords = ["Linux"]
categories = ["Technology"]
+++
### migration
Really migrate this image into the inner intranet, without any internet
connection.    

### Registry Changing
You have to comment the proxy definition, or your registry instance will
restart frequently, thus your dind won't get working using registry.    

```
# vim /root/data/config.yml
	#proxy:
		# remoteurl: https://registry-1.docker.io
# docker restart docker-registry-proxy-2
```
### systemd definition
Define following two systemd units:    

```
# vim /etc/systemd/system/playwithdocker.service 
[Unit]
Description=playwithdocker
After=docker.service
Requires=docker.service

[Service]
Environment=GOPATH=/root/go/
ExecStart=/usr/bin/docker-compose -f /root/go/src/github.com/play-with-docker/play-with-docker/docker-compose.yml up -d

[Install]
WantedBy=multi-user.target
```
This unit will start blog service automatically.    

```
# vim /etc/systemd/system/playwithdockerblog.service 
[Unit]
Description=playwithdockerblog
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/bin/docker-compose -f /root/Code/play-with-docker.github.io/docker-compose.yml up -d

[Install]
WantedBy=multi-user.target
# systemctl enable playwithdocker.service
# systemctl enable playwithdockerblog.service
```
Next time the service will automatically start.    

### Offline CSS/js
bootstrap fonts:   

```
# wget https://github.com/twbs/bootstrap/archive/v3.3.7.zip
# unzip bootstrap-3.3.7.zip
# cd fonts
# mkdir ~/Code/play-with-docker.github.io/_site/fonts/
# cp * ~/Code/play-with-docker.github.io/_site/fonts/
```
Then your image will display correctly.    

![/images/2018_04_08_16_15_38_518x362.jpg](/images/2018_04_08_16_15_38_518x362.jpg)

### Google Fonts
Download the Fonts description from the website, then put all of the related
fonts under your local folder.    

### dnsmasq
Download the rpm package via:    

```
# yum install yum-plugin-downloadonly
# yum reinstall --downloadonly --downloaddir=/root/rpms dnsmasq
```
Transfer the package to intranet and install it. Then edit the configuration
file of dnsmasq:    

```
# vim /etc/dnsmasq.conf
address=/192.192.189.114/192.192.189.114
# systemclt enable dnsmasq && systemctl start dnsmasq
```

