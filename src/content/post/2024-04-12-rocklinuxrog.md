+++
title= "rocklinuxrog"
date = "2024-04-12T11:33:48+08:00"
description = "rocklinuxrog"
keywords = ["Technology"]
categories = ["Technology"]
+++
After installation, change the enpxxx to eth0 via:    

```
 grubby --set-default /boot/vmlinuz-5.14.0-362.8.1.el9_3.x86_64 
 grubby --args="net.ifnames=0" --update-kernel="$(grubby --default-kernel)"
 grubby --args="biosdevname=0" --update-kernel="$(grubby --default-kernel)"
```
Install mono for using fog client:     

```
yum install -y epel-release
sed -e 's|^metalink=|#metalink=|g' \
         -e 's|^#baseurl=https\?://download.fedoraproject.org/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -e 's|^#baseurl=https\?://download.example/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -i.bak \
         /etc/yum.repos.d/epel{,-testing}.repo
yum makecache
yum install -y mono-complete
```
Download the fog client `SmartInstaller.exe`, then:    

```
sudo mono SmartInstaller.exe
...
hhhhhh.owgouwogwoegow.gowugou
...
```
Then you have to enable the ethtool service:      

```
# cat /etc/systemd/system/wol.service 
[Unit]
Description=Enable Wake On Lan

[Service]
Type=oneshot
ExecStart = /sbin/ethtool --change eth0 wol g

[Install]
WantedBy=basic.target
# systemctl enable wol
```
Start and enable:    

```
systemctl enable FOGService
systemctl start FOGService
```
