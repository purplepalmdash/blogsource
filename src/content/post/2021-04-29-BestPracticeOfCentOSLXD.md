+++
title= "BestPracticeOfCentOSLXD"
date = "2021-04-29T10:58:13+08:00"
description = "BestPracticeOfCentOSLXD"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 文档目的
文档旨在针对在CentOS 7操作系统上安装、配置及运行LXD提供最佳实践。   

### 2. 环境准备
基于快速验证的目的，本文档基于虚拟机搭建，验证机配置如下:
* 操作系统： `CentOS Linux release 7.6.1810 (Core) `, 最小化安装
* 硬件配置: 36核心，246GB内存， 500G磁盘分区
* 软件配置: 
* 网络配置: 192.168.100.10/24, 网关192.168.100.1

访问方式(这里提供如何从办公网络直达验证机)

### 3. 环境搭建
离线情况下，配置内网源后，执行以下命令安装:    

```
# yum install -y snapd net-tools vim
# systemctl enable --now snapd.socket
```
解压离线安装文件:    

```
# tar xzvf lxcimages.tar.gz ; tar xzvf snap.tar.gz
```
进入到snap目录下安装snap:    

```
# snap ack core_10958.assert ; snap ack core18_1997.assert; snap ack lxd_20211.assert
# snap install core_10958.snap; snap install core18_1997.snap; snap install lxd_20211.snap
```
更改内核参数后，重启机器:   

```
$ grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"
$ grubby --args="namespace.unpriv_enable=1" --update-kernel="$(grubby --default-kernel)"
$ sudo sh -c 'echo "user.max_user_namespaces=3883" > /etc/sysctl.d/99-userns.conf'
# reboot
```
创建snap目录并添加运行权限:    

```
# ln -s /var/lib/snapd/snap /snap
# usermod -a -G lxd roo
# newgrp lxd
# id
uid=0(root) gid=994(lxd) groups=994(lxd),0(root)
```
此时需要退出终端后重新登录终端，方可使用`lxc`相关命令.   

初始化`lxd`环境:    

```
[root@lxdpaas ~]# lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (btrfs, dir, lvm, ceph) [default=btrfs]: dir
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
Would you like the LXD server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
```
此时应无任何镜像，接下来手动导入镜像:    

```
# cd lxcimages
# lxc image  import meta-50030de846c046680faf34f7dc3e60284e31f5aab38dfd19c94a2fd1bf895d0c.tar.xz 50030de846c046680faf34f7dc3e60284e31f5aab38dfd19c94a2fd1bf895d0c.squashfs --alias centos7
Image imported with fingerprint: 50030de846c046680faf34f7dc3e60284e31f5aab38dfd19c94a2fd1bf895d0c
# lxc image list
+---------+--------------+--------+----------------------------------+--------------+-----------+---------+------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |           DESCRIPTION            | ARCHITECTURE |   TYPE    |  SIZE   |         UPLOAD DATE          |
+---------+--------------+--------+----------------------------------+--------------+-----------+---------+------------------------------+
| centos7 | 50030de846c0 | no     | Centos 7 x86_64 (20210428_07:08) | x86_64       | CONTAINER | 83.46MB | Apr 29, 2021 at 4:53am (UTC) |
+---------+--------------+--------+----------------------------------+--------------+-----------+---------+------------------------------+

```
### 4. lxc操作实练
启动一个lxc 实例:    

```
# lxc launch centos7 db1
Creating db1
Starting db1              
```
进入运行中的实例:    

```
# lxc exec db1 bash
[root@db1 ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
```
启动第二个名为`db2`的实例：    

```
[root@lxdpaas lxcimages]# lxc launch centos7 db2
Creating db2
Starting db2                              
[root@lxdpaas lxcimages]# lxc exec db2 bash
[root@db2 ~]#
```
查看运行中的容器实例:    

```
# lxc ls
+------+---------+-----------------------+----------------------------------------------+-----------+-----------+
| NAME |  STATE  |         IPV4          |                     IPV6                     |   TYPE    | SNAPSHOTS |
+------+---------+-----------------------+----------------------------------------------+-----------+-----------+
| db1  | RUNNING | 10.159.107.72 (eth0)  | fd42:45a:636c:6e69:216:3eff:fe81:347e (eth0) | CONTAINER | 0         |
+------+---------+-----------------------+----------------------------------------------+-----------+-----------+
| db2  | RUNNING | 10.159.107.125 (eth0) | fd42:45a:636c:6e69:216:3eff:fe53:754 (eth0)  | CONTAINER | 0         |
+------+---------+-----------------------+----------------------------------------------+-----------+-----------+
```

停止/删除运行中的容器:    

```
[root@lxdpaas lxcimages]# lxc stop db1
[root@lxdpaas lxcimages]# lxc stop db2
[root@lxdpaas lxcimages]# lxc delete db1
[root@lxdpaas lxcimages]# lxc delete db2
[root@lxdpaas lxcimages]# lxc ls
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
```

定制化:    

```
# lxc launch c75dhclient k2
# lxc exec k2 /bin/bash
dhclient eth0
vi /etc/yum.repos.d/kkk.repo
yum makecache
yum install -y vim net-tools
exit
# lxc ls | grep k2
| k2   | RUNNING | 10.159.107.248 (eth0) | fd42:45a:636c:6e69:216:3eff:fea0:2c33 (eth0) | CONTAINER | 0         |
```
导出当前镜像：   

```
[root@lxdpaas ~]# mkdir export
[root@lxdpaas ~]# cd export/
[root@lxdpaas export]# lxc stop k2
[root@lxdpaas export]# lxc publish k2 --alias centos75withvim
Instance published with fingerprint: 7301c7d85d4d56ebcae117aa79cf88868c4821dedb22e641fe66d05cab6599f2
[root@lxdpaas export]# lxc image list
+-----------------+--------------+--------+----------------------------------+--------------+-----------+----------+------------------------------+
|      ALIAS      | FINGERPRINT  | PUBLIC |           DESCRIPTION            | ARCHITECTURE |   TYPE    |   SIZE   |         UPLOAD DATE          |
+-----------------+--------------+--------+----------------------------------+--------------+-----------+----------+------------------------------+
| c75dhclient     | 3a063c11b987 | no     |                                  | x86_64       | CONTAINER | 381.84MB | Apr 29, 2021 at 8:06am (UTC) |
+-----------------+--------------+--------+----------------------------------+--------------+-----------+----------+------------------------------+
| centos7         | 50030de846c0 | no     | Centos 7 x86_64 (20210428_07:08) | x86_64       | CONTAINER | 83.46MB  | Apr 29, 2021 at 4:53am (UTC) |
+-----------------+--------------+--------+----------------------------------+--------------+-----------+----------+------------------------------+
| centos75withvim | 7301c7d85d4d | no     |                                  | x86_64       | CONTAINER | 420.72MB | Apr 29, 2021 at 8:23am (UTC) |
+-----------------+--------------+--------+----------------------------------+--------------+-----------+----------+------------------------------+
[root@lxdpaas export]# lxc image export centos75withvim .
Image exported successfully!           
[root@lxdpaas export]# ls
7301c7d85d4d56ebcae117aa79cf88868c4821dedb22e641fe66d05cab6599f2.tar.gz
```
测试:    

```
# lxc launch centos75withvim test1
Creating test1
Starting test1                             
[root@lxdpaas export]# lxc exec test1 /bin/bash
[root@base ~]# dhclient eth0
[root@base ~]# which vim
/usr/bin/vim
[root@base ~]# which ifconfig
/usr/sbin/ifconfig

```
有关数据库的更改:    

```
 yum install -y mariadb-server
 systemctl enable mariadb
```

### 5. 资源隔离
制作benchmark容器:     

```
$ lxc launch centos7 bench -c security.privileged=true

	# yum install -y epel-release; yum install -y stress
	# yum install which
	# which stress
        # shutdown -h now
$ lxc publish bench --alias bench
$ lxc launch bench k1
$ lxc exec k1 /bin/bash
    stress --cpu 5
```

此时可以看到，宿主机上的5个cpu跑满：    

![/images/2021_04_29_17_55_21_884x154.jpg](/images/2021_04_29_17_55_21_884x154.jpg)


设置CPU限制:    

```
# lxc config set  bench limits.cpu 2
```
即便容器中的进程未变，但是主机上可以看到，只有两个CPU跑满:    

![/images/2021_04_29_17_56_10_898x161.jpg](/images/2021_04_29_17_56_10_898x161.jpg)

对内存的使用规则是同样的。    

### z. 定制化
为了适配用户习惯，做了以下修改:   

```
# yum install -y mate-desktop xrdp mate* gnome-terminal firefox wqy* evince
# echo mate-session>/root/.Xclients
# chmod 777 /root/.Xclients
# systemctl start xrdp
# systemctl enable xrdp
```
外部需要做iptables转发:    

```
$ sudo iptables -D FORWARD -o virbr0 -j REJECT --reject-with icmp-port-unreachable
$ sudo iptables -D FORWARD -i virbr0 -j REJECT --reject-with icmp-port-unreachable
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 13389 -j DNAT --to-destination  192.168.100.10:3389
$ sudo iptables -t nat -A POSTROUTING -p tcp -d 192.168.100.10 --dport 3389 -j SNAT --to-source 10.50.208.147
```

外部的centos7机器上，因为升级了内核的关系，需要用如下命令开始运行:    

```
lxc launch images:centos/7 blah -c security.privileged=true
```

当前制作的centos7.5容器似乎不能满足lxc的功能?   
