+++
title= "SecurityEnhancement"
date = "2022-05-24T08:57:55+08:00"
description = "SecurityEnhancement"
keywords = ["Technology"]
categories = ["Technology"]
+++
nessus整改策略
### 1. SSH SHA-1 HMAC Algorithms Enabled
问题:   

![/images/2022_05_24_08_58_34_930x453.jpg](/images/2022_05_24_08_58_34_930x453.jpg)

原因:    
sshd服务器开启了hmac-sha1，需要在sshd配置文件中将其关闭并重启, Ubuntu为例整改步骤如下:     

列出所有支持的MAC算法:    

```
# sshd -T | grep macs
macs umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1
```
将上述输出的条目，去掉`hmac-sha1`后，加入到sshd配置文件:        

```
# vim /etc/ssh/sshd_config
........
macs umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512
# systemctl restart ssh && systemctl restart sshd
```
重启后，检查是否移除mac:   

```
# sshd -T | grep macs
macs umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512
```
### 2. MEDIUMUnencrypted Telnet Server
关闭23端口的xinetd服务:    

```
# sudo netstat -anp | grep 23
# sudo systemctl stop xinetd
# sudo systemctl disable xinetd
# sudo netstat -anp | grep 23
```
### 3. SSH Weak Key Exchange Algorithms Enabled
Info:    

```
The following weak key exchange algorithms are enabled : 

  diffie-hellman-group-exchange-sha1
  diffie-hellman-group1-sha1
```
检测、更改配置:       

```
# sshd -T | grep diffie
kexalgorithms curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha256,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
# vim /etc/ssh/sshd_config
kexalgorithms curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256,diffie-hellman-group14-sha1
```
Restart the sshd via `systemctl restart sshd`

### 4. SSH Server CBC Mode Ciphers Enabled
检测:      

```
# sshd -T |grep ciphers
ciphers chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,aes128-cbc,aes192-cbc,aes256-cbc,blowfish-cbc,cast128-cbc,3des-cbc

```

更改配置并重启
```
# vim /etc/ssh/sshd_config
Ciphers aes128-ctr,aes192-ctr,aes256-ctr
```
以上修改效果(整改前):    

![/images/2022_05_24_09_49_30_1323x496.jpg](/images/2022_05_24_09_49_30_1323x496.jpg)

整改后:   


