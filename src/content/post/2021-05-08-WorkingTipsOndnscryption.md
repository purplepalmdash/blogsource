+++
title= "WorkingTipsOndnscryption"
date = "2021-05-08T17:28:12+08:00"
description = "WorkingTipsOndnscryption"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Working Environment
Centos 7.9, vm , 8core, 16G.   

### Installation
Install dnsmasq:   

```
# sudo yum install -y dnsmasq
```
Install dnscrypt-proxy:    

```
# sudo yum install -y dnscrypt-proxy
```
Wget the chinadns configuration file:    

```
# wget https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/accelerated-domains.china.conf
# mv accelerated-domains.china.conf /etc/dnsmasq.d/accelerated-domains.china.conf
```
you can replace the `114.114.114.114` via your own dns(china intranet dns).    

### Configuration
Configure dnsmasq:    

```
# vim /etc/dnsmasq.conf
listen-address=127.0.0.1
no-resolv
conf-dir=/usr/local/etc/dnsmasq.d
server=127.0.0.1#5300
interface=lo
bind-interfaces
```

Configure dnscrypt-proxy:    

```
# vim /etc/dnscrypt-proxy/dnscrypt-proxy.toml
     # 监听5300端口
     listen_addresses = ['127.0.0.1:5300', '[::1]:5300']
     # 使用下面3个公开的DNS服务
     server_names = ['google', 'cloudflare', 'cloudflare-ipv6']
     # 如果找不到合适的公开DNS服务，则使用下面的DNS服务
     fallback_resolvers = ['9.9.9.9:53', '8.8.8.8:53']
     # 担心这些DNS请求被墙，设置使用代理发送DNS请求
     force_tcp = true
     proxy = 'socks5://127.0.0.1:1086'
```

Configure `/etc/resolv.conf` for using `127.0.0.1`:    

```
nameserver 127.0.0.1
```
### privoxy
In centos 7.9. don't install this package from epel, download the source code from internet and compile it:    

```
$ privoxy  --version
Privoxy version 3.0.28 (https://www.privoxy.org/)
```
make sure you have specify the gfwlist.  
