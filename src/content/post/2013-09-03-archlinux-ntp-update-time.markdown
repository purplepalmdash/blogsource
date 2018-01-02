---

comments: true
date: 2013-09-03T00:00:00Z
title: ArchLinux Ntp update time
url: /2013/09/03/archlinux-ntp-update-time/
---

1\. 安装ntpd
```
$ pacman -S ntpd
```

2\. 配置ntp服务器
```
$ vim /etc/ntp.conf
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst
```

3\. 同步时间
```
$ sudo ntpd -gq
```

4\. 加入守护进程运行
```
$ sudo systemctl enable ntpd
```
