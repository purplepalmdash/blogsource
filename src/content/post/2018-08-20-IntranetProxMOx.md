+++
title = "内网搭建proxmox"
date = "2018-08-20T16:42:20+08:00"
description = "内网搭建proxmox"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 环境准备
Iso使用官方下载的`proxmox-ve_5.2-1.iso`, CPU/内存配置为16核64G。    
硬盘配置为- 系统:200G, Ceph存储, 600G    
一共三台机器，均为虚拟机，位于不同的物理机器上，这点非常重要，如果处于同一机器上，则在线迁移虚拟机容易出现错误，具体表现为，虚拟机迁移完毕以后，被迁移出的那台机器节点将失去反应，节点无法登录。    

CPU我们通过`host-passthrough`下发到虚拟机里。    

### IP地址配置
节点1(zzz_proxmox_127), 位于127服务器，ip为10.33.34.27, hostname为promox127.    
节点2(zzz_proxmox_128), 位于128服务器，ip为10.33.34.28, hostname为promox128.    
节点3(zzz_proxmox_129), 位于129服务器，ip为10.33.34.29, hostname为promox129.    

用于Ceph的地址暂时不配置。

### 开启multicast
proxmox需要各个节点的multicast为可用状态，而默认的virt-manager禁用了该选项，我们使用以下命令来开启虚拟机上的multicast.    

```
for dev in `ls /sys/class/net/ | grep macvtap`; do
    ip link set dev $dev allmulticast on
  done
```

### 建立集群
浏览器访问`https://10.33.34.27:8006`，选择语言后，页面如下:    

![/images/2018_08_20_17_37_10_734x555.jpg](/images/2018_08_20_17_37_10_734x555.jpg)

现在只有一个节点:    

![/images/2018_08_20_17_37_31_576x326.jpg](/images/2018_08_20_17_37_31_576x326.jpg)

27上运行命令, create创建出一个集群，而status则是检查其状态:     

```
# pvecm create firstcluster
# pvecm status
```

28/29上分别运行:    

```
# pvecm add 10.33.34.27
```

添加完毕后的集群如下:    

![/images/2018_08_20_17_40_17_476x330.jpg](/images/2018_08_20_17_40_17_476x330.jpg)


### Ceph
配置IP地址:    

```
# from /etc/network/interfaces
auto eth2
iface eth2 inet static
  address  10.10.10.1
  netmask  255.255.255.0
```
修改pveceph的源:    

```
# vi /usr/share/perl5/PVE/CLI/pveceph.pm
deb .......
# pveceph install --version luminous
```

添加存储:    

![/images/2018_08_21_12_13_12_666x301.jpg](/images/2018_08_21_12_13_12_666x301.jpg)

创建一个pool,   

![/images/2018_08_21_12_13_39_301x275.jpg](/images/2018_08_21_12_13_39_301x275.jpg)

创建完毕后:   

![/images/2018_08_21_12_14_03_915x372.jpg](/images/2018_08_21_12_14_03_915x372.jpg)

### 虚拟机
拷贝安装文件`ubuntu-16.04.2-server-amd64.iso`到`/var/lib/vz/template/iso`下，
在27机器上， 然后创建虚拟机。    

![/images/2018_08_21_12_17_24_460x214.jpg](/images/2018_08_21_12_17_24_460x214.jpg)

选择ISO：    

![/images/2018_08_21_12_17_42_668x184.jpg](/images/2018_08_21_12_17_42_668x184.jpg)

选择硬盘:    

![/images/2018_08_21_12_18_07_638x212.jpg](/images/2018_08_21_12_18_07_638x212.jpg)

选择刚创建的虚拟机，点击`启动`:   

![/images/2018_08_21_12_18_51_852x563.jpg](/images/2018_08_21_12_18_51_852x563.jpg)

### Issue
嵌套虚拟化对内核版本的影响，因内网的机器运行的操作系统内核版本较为陈旧，相信可能会有问题。后期将新装服务器来进行。    

新装物理服务器，dhcp得到同一网段地址，而后将继续proxmox的测试。    
