+++
title= "AutoPingOpenWRT"
date = "2022-04-24T08:39:24+08:00"
description = "AutoPingOpenWRT"
keywords = ["Technology"]
categories = ["Technology"]
+++
坐标广州，家里申请的公网IP地址如果隔段时间不使用的话会被回收，每次回收后都要打电话弄回来，如何自动的让公网IP不被回收呢，就需要每次从外部Ping或者Get某个服务。下面是实现步骤。    

### 前置条件
Call 10000, 人工服务，要求光猫上绑定公网IP,理由是家里装监控需要用到。

### 光猫设置
光猫上设置端口映射，如12222端口映射到某台OpenWRT的22端口.   

### OpenWRT设备设置
OpenWRT的dropbear设置ssh key:    

```
cd /root/.ssh
dropbearkey -t rsa -f ~/.ssh/id_rsa
dropbearkey -y -f ~/.ssh/id_rsa | grep "^ssh-rsa " >> authorized_keys
```
拷贝`authorized_keys`到远端vps的`authorized_keys`文件中，由此实现无密码登陆。   

### 撰写无码验证脚本
撰写`/overlay/remote.sh`文件如下:    

```
#!/bin/sh
pIP=`wget -qO - http://icanhazip.com`
ssh -i /root/.ssh/id_rsa -p2xxxx root@1x.xx.xx.xx "ssh -p12222 -o StrictHostKeyChecking=no root@$pIP 'date'| tee /root/log.txt"
```
其中`1x.xx.xx.xx`为远端vps地址, `2xxxx`为远端vps的ssh端口。    

```
# chmod 777 /overlay/remote.sh
```

### OpenWRT crontab
编辑crontab任务如下:   

```
# crontab -e
@hourly /overlay/remote.sh
```

### 结论
由此实现了OpenWRT路由器上发起的每一个小时通过远程vps联会自己的公网地址上暴露的IP端口，由此可以保证公网IP永不过期.    
