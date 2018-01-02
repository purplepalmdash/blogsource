+++
keywords = ["linux"]
description = "Ubuntu16.04 systemd startup program configuration"
title = "Ubuntu16.04 systemd 启动程序配置"
date = "2017-03-04T16:11:19+08:00"
categories = ["Linux"]

+++
### 目的
最近在调查一个很棘手的问题，某司的IAAS平台CloudStack上有一个重置虚拟机密码
的功能，在系统创建的时候，可以通过cloud-init的方式给新创建的虚拟机设置新
root密码。然而在比较新的系统上，例如Ubuntu 16.04, 重置密码功能会失败，因而
有了以下的调研。    

### 实验环境
Ubuntu16.04 Vagrant.    

### 问题调研过程
Cloudstack用于重置密码的实现，在编译模板时取决于以下代码:    

```
wget http://download.cloud.com/templates/4.2/bindir/cloud-set-guest-password.in
sudo mv cloud-set-guest-password.in /etc/init.d/cloud-set-guest-password
sudo chmod +x /etc/init.d/cloud-set-guest-password
sudo update-rc.d cloud-set-guest-password defaults 98
```
以上代码的意思在于，下载`cloud-set-guest-password.in`文件，将其加入到系统的启动
目录中，而最后的`update-rc.d`这一行则是将启动顺序为最后(默认99)。    

在sysV的系统上，这样的配置是没有问题的，例如Ubuntu14.04，确实是可以通过
这样的配置来实现虚拟机的密码重置功能的。然而在ubuntu16.04系统上，重置密码会失败。    

查看代码，看到代码里有很多的`logger -t "cloud"
"xxxxx"`之类的代码，可以看到这个脚本在重置密码的时候会写入系统的`/var/log/syslog`文件    

查看`/var/log/syslog`文件，可以看到以下错误信息:    

![/images/2017_03_04_16_22_28_916x199.jpg](/images/2017_03_04_16_22_28_916x199.jpg)

而看到代码中，代码重置依赖于`wget`来得到密码文件，如果网络不通，则会走入到错误
流程。接着往下看log文件，可以看到有dhcp获得IP地址，然而这个过程却出现在`cloud-set-guest-password`
脚本执行之后。    

![/images/2017_03_04_16_31_58_900x651.jpg](/images/2017_03_04_16_31_58_900x651.jpg)

解决方案的关键在于，在通过DHCP获得IP地址以后再执行密码重置进程    


### 激活 ifup-wait-all-auto服务
Ubuntu使用`/etc/network/interface`文件用于配置网络，所以我们需要一个systemd服务
用于查看每个网络设备的状态。我们先查看下在系统的systemd配置目录下是否有`ifup-wait-all-auto`
定义文件(在Ubuntu 15.04系统中由ifupdown包安装).
如果没有的话，则创建一个`/lib/systemd/system/ifup-wait-all-auto.service`文件，并写入
以下内容:    

```
[Unit]
Description=Wait for all "auto" /etc/network/interfaces to be up for network-online.target
Documentation=man:interfaces(5) man:ifup(8)
DefaultDependencies=no
After=local-fs.target
Before=network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
TimeoutStartSec=2min
ExecStart=/bin/sh -ec '\
  for i in $(ifquery --list --exclude lo --allow auto); do INTERFACES="$INTERFACES$i "; done; \
  [ -n "$INTERFACES" ] || exit 0; \
  while ! ifquery --state $INTERFACES >/dev/null; do sleep 1; done; \
  for i in $INTERFACES; do while [ -e /run/network/ifup-$i.pid ]; do sleep 0.2; done; done'

[Install]
WantedBy=network-online.target
```
这个定义文件虽然是Ubuntu15.04的，但一样适用于Ubuntu16.04. 使能服务:    

```
$ sudo systemctl enable ifup-wait-all-auto.service
```
在使能此服务以后，`network-oneline.target`标志就可以使用了. 任何一个依赖于网络接口可用的
服务，只要加入对此标志的依赖，就可以保证服务在网口可用后启动.    

### 创建cloudpass服务
创建一个名为cloudpass的服务，服务定义文件是`/lib/systemd/system/cloudpasswd.service`:    

```
[Unit]
Description=cloudpass container
Wants=network-online.target
After=network-online.target

[Service]
Type=idle
Restart=always
RemainAfterExit=true
ExecStart=/usr/bin/cloud-set-guest-password
ExecStop=/usr/bin/echo cloudsetguestpasswordDone

[Install]
WantedBy=multi-user.target
```
而`/usr/bin/cloud-set-guest-password`文件则是将以前的密码重置脚本拷贝到`/usr/bin`下。    

使能其服务:    

```
$ sudo systemctl enable cloudpass.service
```

现在重新创建模板，这次创建的模板可以正常重置密码成功.    

### Network-Manager服务
如果涉及到多网络的自动化配置，则可能需要启用Network-Manager，安装:    

```
$ sudo apt-get install -y network-manager
```
现在启用服务，并激活`NetWorkManager-wait-online.service`服务:    

```
$ sudo systemctl enable NetworkManager
```

`192.168.1.69,/home/juju/BuildOneClick__Box_OrQcow2`
