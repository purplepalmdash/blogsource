+++
title= "RunSystemctlWithoutRootPassword"
date = "2023-05-05T12:41:18+08:00"
description = "RunSystemctlWithoutRootPassword"
keywords = ["Technology"]
categories = ["Technology"]
+++
创建一个sudoer规则文件: 

```
$ echo -e "Cmnd_Alias USER_SERVICES = /usr/bin/systemctl start snmpd.service, /usr/bin/systemctl stop snmpd.service, /usr/bin/systemctl restart snmpd.service, /usr/bin/systemctl start p910nd.service, /usr/bin/systemctl stop p910nd.service, /usr/bin/systemctl restart p910nd.service\nctyun ALL=(ALL) NOPASSWD: USER_SERVICES" | sudo tee /etc/sudoers.d/ctyun
```
(另一种方法)也可以手动创建`/etc/sudoers.d/ctyun`文件后键入以下内容:    

```
Cmnd_Alias USER_SERVICES = /usr/bin/systemctl start snmpd.service, /usr/bin/systemctl stop snmpd.service, /usr/bin/systemctl restart snmpd.service, /usr/bin/systemctl start p910nd.service, /usr/bin/systemctl stop p910nd.service, /usr/bin/systemctl restart p910nd.service
ctyun ALL=(ALL) NOPASSWD: USER_SERVICES
```
重启机器，或重新登陆桌面(注销后登陆), 验证: 
   
```
sudo systemctl restart snmpd.service
```
