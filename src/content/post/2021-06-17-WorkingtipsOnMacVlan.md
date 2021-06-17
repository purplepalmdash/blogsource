+++
title= "WorkingtipsOnMacVlan"
date = "2021-06-17T21:19:03+08:00"
description = "WorkingtipsOnMacVlan"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 解决问题
LXD 的macvlan组网方式，共享与主机相同的地址段.   

### 环境
环境如下:    

```
lxd1 eth0 192.168.100.61
lxd2 eth0 192.168.100.62
lxd3 eth0 192.168.100.63
```
创建出的`macvlan`的profile如下:    

```
# lxc profile show macvlan
config: {}
description: Default LXD profile modified for using macvlan
devices:
  eth0:
    nictype: macvlan
    parent: eth0
    type: nic
name: macvlan
```
在lxd2主机上创建实例并检查:    

```
# lxc launch 48f7ccdc7b02 test1 --profile default --profile macvlan
# lxc ls
| test1 | RUNNING | 192.168.100.253 (eth0) |      | CONTAINER | 0         |
# lxc exec test1 bash
ping 192.168.100.61/63都可以
无法ping 192.168.100.62
```
同样在lxd1和lxd3上的容器实例亦无法ping通本机。    

### workaround
虽然macvlan无法Ping通物理机，因为这是由它的设计原理决定的。但是macvlan可以ping通其他的macvlan， 因而我们可以在主机上分别加上一个额外的macvlan专用来与本机上启动的Lxc容器通信：    

以lxd2主机为例，新增一个`mynet`的macvlan网络设备，指定其地址为`192.168.100.72`, 而后添加路由，所有到达其产生的LXC容器实例的流量均经过该mynet设备转发:   

```
ip link add mynet link eth0 type macvlan mode bridge
ip addr add 192.168.100.72 dev mynet
ip link set mynet up
ip route add 192.168.100.253 dev mynet
```
创建完成后，ping本机上的test1容器(192.168.100.253):    

```
PING 192.168.100.253 (192.168.100.253) 56(84) bytes of data.
64 bytes from 192.168.100.253: icmp_seq=1 ttl=64 time=0.088 ms
```
进入容器后亦可ping通主机.    

检查主机上新创建的设备及路由:    

```
# ip addr show mynet
5: mynet@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether ba:2b:e7:86:33:9a brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.72/32 scope global mynet
       valid_lft forever preferred_lft forever
    inet6 fe80::b82b:e7ff:fe86:339a/64 scope link 
       valid_lft forever preferred_lft forever
# ip route 
default via 192.168.100.1 dev eth0 proto static metric 100 
10.230.202.0/24 dev lxdbr0 proto kernel scope link src 10.230.202.1 
192.168.100.0/24 dev eth0 proto kernel scope link src 192.168.100.62 metric 100 
192.168.100.253 dev mynet scope link 
```

### ToDO
如果要用于生产环境的话，需要考虑：    

```
1. 自动化创建mynet设备并绑定一个额外的同网段IP地址。
2. LXC实例创建后，自动在主机层面创建经由mynet设备的路由。
3. LXC销毁后，自动删除该LXC IP对应的路由。
```
