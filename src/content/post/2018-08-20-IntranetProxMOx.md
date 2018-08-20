+++
title = "内网搭建proxmox"
date = "2018-08-20T16:42:20+08:00"
description = "内网搭建proxmox"
keywords = ["Linux"]
categories = ["Linux"]
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

