+++
title= "WorkingTipsOnIncusBasedFogServer"
date = "2024-05-08T09:04:13+08:00"
description = "WorkingTipsOnIncusBasedFogServer"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. dhcpd服务器考虑
#### 1.1 dhcpd在容器内出现的问题
因为incus内部使用的是私有网络，因此一旦将`isc-dhcp-server`启动在容器内，则只能监听容器内部的地址，而一旦更改子网定义为主机网络侧，则会出现dhcpd.conf检查错误:    

```
5月 08 08:58:44 fogincuschinese dhcpd[2532]: No subnet declaration for eth0 (10.147.148.100).
5月 08 08:58:44 fogincuschinese dhcpd[2532]: ** Ignoring requests on eth0.  If this is not what
5月 08 08:58:44 fogincuschinese dhcpd[2532]:    you want, please write a subnet declaration
5月 08 08:58:44 fogincuschinese dhcpd[2532]:    in your dhcpd.conf file for the network segment
5月 08 08:58:44 fogincuschinese dhcpd[2532]:    to which interface eth0 is attached. **
5月 08 08:58:44 fogincuschinese dhcpd[2532]: 
5月 08 08:58:44 fogincuschinese dhcpd[2532]: 
5月 08 08:58:44 fogincuschinese dhcpd[2532]: Not configured to listen on any interfaces!
```
出现上述问题的原因在于：容器内eth0为`10.147.148.100`, 而主机侧为`192.168.1.0.24`, 无法写如下的配置文件(diff文件更改了默认dhcpd.conf中的监听网段) :    

```
root@fogincuschinese:~# diff /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.back 
24c24
< subnet 192.168.1.0 netmask 255.255.255.0{
---
> subnet 10.147.148.0 netmask 255.255.255.0{
26c26
<     range dynamic-bootp 192.168.1.50 192.168.1.90;
---
>     range dynamic-bootp 10.147.148.10 10.147.148.254;
29c29
<     option routers 192.168.1.33;
---
>     option routers 10.147.148.1;
31c31
<     next-server 192.168.1.40;
---
>     next-server 10.147.148.100;
```
#### 1.2 dhcpd在主机上的实现
安装:     

```
# apt install -y isc-dhcp-server
```
配置，使能:    

```
$ sudo vim /etc/dhcp/dhcpd.conf 
subnet 192.168.1.0 netmask 255.255.255.0{
    option subnet-mask 255.255.255.0;
    range dynamic-bootp 192.168.1.50 192.168.1.90;
    default-lease-time 21600;
    max-lease-time 43200;
    option routers 192.168.1.33;
    option domain-name-servers 223.5.5.5;
    next-server 192.168.1.40;
}
$ sudo systemctl daemon-reload
$ sudo systemctl start isc-dhcp-server
$ sudo systemctl enable isc-dhcp-server
```
需要对齐fogserver中的配置.    

### 2. tftpd服务
如果直接使用inpus中的容器，则因为tftp的通信协商机制中，会因为使用1024~65535的随机端口和客户端通信，而容器本身比较难搞定proxy, 而导致通信失败。    
因此我们需要将tftpd也从里面搞出来， 在主机上实现。   

但，如果做了这么多以后，还是容器吗？    

所以上面的方式，未必适合

### 3. macvlan网
直接用macvlan直接覆盖。   

注意替换IP：    

```
Change FOG Server IP Address
Procedural Steps
Follow appropriate steps for your Linux distribution to change the OS’s IP address.

Update the ipaddress= field (and other fields if necessary) inside the /opt/fog/.fogsettings file. The .fogsettings file.

Rerun the installer, you’ll need to use –recreate-CA and –recreate-keys keys as the installer provides a certificate with a Common Name based on the ip which will be shipped in the iPxe kernel and failed to load any https resources as the certificate isn’t valid anymore.

Update the IP address inside /tftpboot/default.ipxe (look for the chain line i.e chain https://x.x.x.x/fog/service/ipxe/boot.php##params)

Update the IP address for the storage node on the FOG system where you changed the IP address Web Interface -> Storage Management

Update the IP address on a any master storage node that may reference this FOG server Web Interface -> Storage Management

(For master server) Update the FOG_WEB_HOST value Web Interface -> FOG Configuration -> FOG Settings -> Web Server -> FOG_WEB_HOST

(For master server) Update the FOG_TFTP_HOST value Web Interface -> FOG Configuration -> FOG Settings -> TFTP Server -> FOG_TFTP_HOST

Optionaly if you have configured a dhcpd:

Update IP addresses (fog and gateway) inside the /etc/dhcp/dhcpd.conf.

Don’t forgot to check your /etc/export for nfs server as well as your apache2 configuration as the installer override it.
``` 
### incus 快速启动流程
1. 用户需要安装好incus.    
2. 导入镜像，配置网路。    
3. 快速开出实例，作为Pxe和部署服务器使用。   

Steps:    

```
dash@server:~$ tar xzvf incusdebs.tar.gz
$ sudo chmod 777 -R incuddebs
$ sudo apt install -y incus
sudo adduser dash incus-admin

``` 
init yaml:    

```
config:
  images.auto_update_interval: "0"
networks: []
storage_pools:
- config: {}
  description: ""
  name: default
  driver: dir
profiles:
- config: {}
  description: ""
  devices:
    root:
      path: /
      pool: default
      type: disk
  name: default
projects: []
cluster: null
```
quickly init:     

```
# cat init.sh 
cat <<EOF | incus admin init --preseed
config:
  images.auto_update_interval: "0"
networks: []
storage_pools:
- config: {}
  description: ""
  name: default
  driver: dir
profiles:
- config: {}
  description: ""
  devices:
    root:
      path: /
      pool: default
      type: disk
  name: default
projects: []
cluster: null
EOF

```
Create incusbr0(macvlan):    

```
 incus network create incusbr0 --type=macvlan parent=enp7s0
```
Edit profile default:    

```
cat default.yaml | incus profile edit default
```
Add new profile:    

```
incus profile create nfs-server
    cat nfs-server-profile.yaml | incus profile edit nfs-server
```

另外的机器上导出镜像:     

```
# incus publish fogincuschinese --alias fogAuto
Instance published with fingerprint: 0a4a4299661f19d880a203031b4e7df88996a99be110979975633baf9504b1dc
# incus image export fogAuto .
Image exported successfully!           

```
开始导入镜像:     

```
# ls -l -h 0a4a4299661f19d880a203031b4e7df88996a99be110979975633baf9504b1dc.tar.gz 
-rw-rw-r-- 1 dash dash 857M May  8 09:06 0a4a4299661f19d880a203031b4e7df88996a99be110979975633baf9504b1dc.tar.gz
root@server:/home/dash# incus image import 0a4a4299661f19d880a203031b4e7df88996a99be110979975633baf9504b1dc.tar.gz --alias fogAuto
Image imported with fingerprint: 0a4a4299661f19d880a203031b4e7df88996a99be110979975633baf9504b1dc
root@server:/home/dash# incus image list
+---------+--------------+--------+----------------------------------------+--------------+-----------+-----------+----------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |              DESCRIPTION               | ARCHITECTURE |   TYPE    |   SIZE    |     UPLOAD DATE      |
+---------+--------------+--------+----------------------------------------+--------------+-----------+-----------+----------------------+
| fogAuto | 0a4a4299661f | no     | Debian bookworm amd64 (20240506_05:24) | x86_64       | CONTAINER | 856.42MiB | 2024/05/08 09:08 UTC |
+---------+--------------+--------+----------------------------------------+--------------+-----------+-----------+----------------------+

```
开始开出第一个实例:      

```
# LANG=zh_CN.UTF-8 incus launch fogAuto fogInstance  -p nfs-server -p default -c security.privileged=true -c raw.apparmor="mount fstype=rpc_pipefs, mount fstype=nfsd,"
Launching fogInstance
root@server:/home/dash# incus  list        
+-------------+---------+---------------------+------+-----------+-----------+
|    NAME     |  STATE  |        IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------+---------+---------------------+------+-----------+-----------+
| fogInstance | RUNNING | 192.168.1.44 (eth0) |      | CONTAINER | 0         |
+-------------+---------+---------------------+------+-----------+-----------+
root@server:/home/dash# incus exec fogInstance bash
root@fogInstance:~# 

```
进入到实例里开始重新部署为`192.168.1.46`:    

```
root@fogInstance:~# cd regen/
root@fogInstance:~/regen# ls
1_regen.sh  2_reinstall.sh  cn-fogproject-master  inventoy.ini	mysql.sh  regen.yml  templates
root@fogInstance:~/regen# vim inventoy.ini 
root@fogInstance:~/regen# ./1_regen.sh 

PLAY [Write fogsettings] *********************************************************************************************************************************************************************

...
这里需要重启一次容器实例，非常快:   

root@server:/home/dash# incus exec fogInstance bash
root@fogInstance:~# cd regen/
root@fogInstance:~/regen# ./2_reinstall.sh 

```
push 镜像到相应位置:    

```
incus file push -pr idvnext/ fogInstance/images/idvnext/
```
接下来就可以愉快的玩耍了。

### 综合
传递文件:     

```
$ scp -r hostincus/ dash@192.168.1.38:~
```
