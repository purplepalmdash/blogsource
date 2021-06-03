+++
title= "EnableHostNetworkInLXD"
date = "2021-06-03T06:35:28+08:00"
description = "EnableHostNetworkInLXD"
keywords = ["Technology"]
categories = ["Technology"]
+++
lxc/lxd lacks of using host network like `docker run -it --net host`, following are the steps on how to enable this feature:    

注：使用host网络模式会使得主机层面的网络增强在容器层面被绕过，等同于完全放开了网络安全策略，被攻击面增大。

建议谨慎使用。


### lxd开启hostNetwork
将默认额lxd profile导入到文件中:   

```
# lxc profile show default>host
```
编辑文件，去掉关于网卡的配置(前置`-`的行需要被删除):   

```
# vim host
config:
  security.secureboot: "false"
description: Default LXD profile
devices:
-  eth0:
-    name: eth0
-    network: lxdbr0
-    type: nic
  root:
    path: /
    pool: default
    type: disk
name: default
used_by:
- /1.0/instances/centos
```
删除后保存文件，创建一个名称为`hostnetwork`的`profile`，用修改过的配置文件定义之:   

```
lxc profile create hostnetwork
lxc profile edit hostnetwork<host
```
创建一个新实例:   

```
# lxc init adcfe657303d ubuntu1
```
指定`hostnetwork`的profile为其启动所需的profile:    

```
# lxc profile assign ubuntu1 hostnetwork
```
创建一个`raw.lxc`的外部配置文件:    

```
# vim /root/lxc_host_network_config
lxc.net.0.type = none
```
配置该lxc实例的属性(`raw.lxc`及特权容器), 在某些系统上可能不需要特权容器, 设置完毕后启动:   

```
lxc config set ubuntu1 raw.lxc="lxc.include=/root/lxc_host_network_config"
lxc config set ubuntu1 security.privileged=true
lxc start ubuntu1
```
检查lxc实例所拥有的网络地址空间:    

```
# lxc ls
lxc ls
+---------+---------+---------------------------+--------------------------------------------------+-----------------+-----------+
|  NAME   |  STATE  |           IPV4            |                       IPV6                       |      TYPE       | SNAPSHOTS |
+---------+---------+---------------------------+--------------------------------------------------+-----------------+-----------+
| centos  | RUNNING | 10.225.0.168 (enp5s0)     | fd42:1cac:64d0:f018:547d:b31b:251b:381e (enp5s0) | VIRTUAL-MACHINE | 0         |
+---------+---------+---------------------------+--------------------------------------------------+-----------------+-----------+
| ubuntu1 | RUNNING | 192.192.189.1 (virbr1)    | fd42:1cac:64d0:f018::1 (lxdbr0)                  | CONTAINER       | 0         |
|         |         | 192.168.122.1 (virbr0)    | 240e:3b5:cb5:abf0:36d4:30d3:285b:862e (enp3s0)   |                 |           |
|         |         | 192.168.1.222 (enp3s0)    |                                                  |                 |           |
|         |         | 172.23.8.165 (ztwdjmv5j3) |                                                  |                 |           |
|         |         | 172.17.0.1 (docker0)      |                                                  |                 |           |
|         |         | 10.33.34.1 (virbr2)       |                                                  |                 |           |
|         |         | 10.225.0.1 (lxdbr0)       |                                                  |                 |           |
+---------+---------+---------------------------+--------------------------------------------------+-----------------+-----------+
```
进入到网络中检查所有网卡:   

```
# lxc exec ubuntu1 bash
root@ubuntu1:~# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 74:d4:35:6a:84:19 brd ff:ff:ff:ff:ff:ff
3: ztwdjmv5j3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 2800 qdisc fq_codel state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 0e:d0:a9:d8:58:2a brd ff:ff:ff:ff:ff:ff
4: wlp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DORMANT group default qlen 1000
    link/ether 40:e2:30:30:1e:ee brd ff:ff:ff:ff:ff:ff
5: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:ea:66:bb brd ff:ff:ff:ff:ff:ff
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:d0:f9:2f:53 brd ff:ff:ff:ff:ff:ff
7: lxdbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:b7:04:a9 brd ff:ff:ff:ff:ff:ff
8: virbr1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:88:da:f3 brd ff:ff:ff:ff:ff:ff
9: virbr2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:7a:73:8b brd ff:ff:ff:ff:ff:ff
10: tapeac054b8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master lxdbr0 state UP mode DEFAULT group default qlen 1000
    link/ether fe:25:d8:30:90:f6 brd ff:ff:ff:ff:ff:ff
11: veth43eb373c@veth5f86c132: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:cb:fe:fc brd ff:ff:ff:ff:ff:ff
12: veth5f86c132@veth43eb373c: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue master lxdbr0 state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether 9e:b0:74:e7:fa:86 brd ff:ff:ff:ff:ff:ff
19: veth8d89fdf9@vethb3a2bfad: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:b3:06:c7 brd ff:ff:ff:ff:ff:ff
20: vethb3a2bfad@veth8d89fdf9: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue master lxdbr0 state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether da:c7:ab:6b:90:7f brd ff:ff:ff:ff:ff:ff
```


### lxc 开启hostNetwork
使用lxc创建一个容器:   

```
lxc-create -n test -t download -- --dist ubuntu --release focal --arch amd64
```


编辑 /var/lib/lxc/test/config. 删除所有关于network的行, 添加 "lxc.network.type = none"

启动容器:   

```
lxc-start -n test
```

检查lxc网络设置:   

```
lxc-ls -f


NAME STATE   AUTOSTART GROUPS IPV4                                                                                          IPV6                                                          UNPRIVILEGED 
test RUNNING 0         -      10.225.0.1, 10.33.34.1, 172.17.0.1, 172.23.8.165, 192.168.1.222, 192.168.122.1, 192.192.189.1 240e:3b5:cb5:abf0:36d4:30d3:285b:862e, fd42:1cac:64d0:f018::1 false   
```

进入到容器后检查网卡:    

```

➜  ~ lxc-attach test  
# /bin/bash
root@test:~# ip addr
这里可以看到主机上所有的接口地址

```

