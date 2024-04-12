+++
title= "LocalDNSOnFog"
date = "2024-04-12T10:40:10+08:00"
description = "LocalDNSOnFog"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install dnsmasq via:     

```
sudo apt install -y dnsmasq
```
Edit the configuration file(`vim /etc/dnsmasq.conf`):      

```
...

# 不允许 dnsmasq 通过轮询 /etc/resolv.conf 或者其他文件来获取配置的改变，则取消注释。
#no-poll
# 向上游所有服务器查询
all-servers
# 启用转发循环检测
dns-loop-detect
# 重启后清空缓存
clear-on-reload
# 完整域名才向上游服务器查询，如果是主机名仅查找 hosts 文件
domain-needed

# 指定 dnsmasq 默认查询的上游服务器，此处以 Google Public Dns 为例。
server=223.5.5.5

# no-hosts, 默认情况下这是注释掉的，dnsmasq 会首先寻找本地的 hosts 文件，再去寻找缓存下来的域名，最后去上级 Dns 服务器中寻找；而 addn-hosts 可以使用额外的 hosts 文件。
# Dns 解析 hosts 时对应的 hosts 文件，对应 no-hosts
addn-hosts=/etc/hosts
# Dns 缓存大小，Dns 解析条数
cache-size=1024
# 不缓存未知域名缓存，默认情况下 dnsmasq 会缓存未知域名并直接返回客户端
no-negcache
# 指定 Dns 同时查询转发数量
dns-forward-max=1000

# 增加一个域名，强制解析到所指定的地址上，强行指定 domain 的 IP 地址
address=/hhhhhh.ctyun.net.cn/192.168.1.22
...
```
Test via:      

```
dig @192.168.1.22 hhhhhh.ctyun.net.cn
dig @192.168.1.22 www.baidu.com
```
Then edit the dhcpd.conf:     

```
# vim /etc/dhcp/dhcpd.conf
....

option domain-name-servers 192.168.1.22;
....
```
