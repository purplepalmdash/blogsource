+++
title= "WorkingTipsOnCross"
date = "2023-11-13T11:22:09+08:00"
description = "WorkingTipsOnCross"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. dnscrypt-proxy
Uninstall `systemd-resolved` then install `dnscrypt-proxy`:    

```
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
sudo apt install -y dnscrypt-proxy
sudo systemctl start dnscrypt-proxy
```
Examine its running:      

```
$ sudo netstat -anp | grep ":53" | grep "127.0.2.1"
tcp        0      0 127.0.2.1:53            0.0.0.0:*               LISTEN      1/init              
udp        0      0 127.0.2.1:53            0.0.0.0:*                           1/init       
```
Configure default `dnscrypt-proxy.toml` file:    

```
$ cat /etc/dnscrypt-proxy/dnscrypt-proxy.toml 
# Empty listen_addresses to use systemd socket activation
listen_addresses = ['0.0.0.0:5533']
server_names = ['alidns-doh','tuna-doh-ipv4']
cache_size = 4096
cache_min_ttl = 2400
cache_max_ttl = 86400
cache_neg_min_ttl = 60
cache_neg_max_ttl = 600

[query_log]
  file = '/var/log/dnscrypt-proxy/query.log'
......
```
Copy the `dnscrypt-proxy.toml` file for foreign dns query:    

```
$ sudo cp /etc/dnscrypt-proxy/dnscrypt-proxy.toml /etc/dnscrypt-proxy/dnscrypt-proxy-foreign.toml
```
Change the content of this foreign content:     

```
$ cat /etc/dnscrypt-proxy/dnscrypt-proxy-foreign.toml 
# Empty listen_addresses to use systemd socket activation
listen_addresses = ['0.0.0.0:25533']
server_names = ['cloudflare','google']
cache_size = 4096
cache_min_ttl = 2400
cache_max_ttl = 86400
cache_neg_min_ttl = 60
cache_neg_max_ttl = 600


[query_log]
  file = '/var/log/dnscrypt-proxy/query1.log'

[nx_log]
  file = '/var/log/dnscrypt-proxy/nx1.log'
......
```
For using this newly created configuration file , create a new systemd item:     

```
$ sudo cp /lib/systemd/system/dnscrypt-proxy.service /lib/systemd/system/dnscrypt-proxy-foreign.service
$ sudo vim /lib/systemd/system/dnscrypt-proxy-foreign.service 
...
ExecStart=/usr/sbin/dnscrypt-proxy -config /etc/dnscrypt-proxy/dnscrypt-proxy-foreign.toml
...
```
Configure this service:    

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable dnscrypt-proxy
$ sudo systemctl enable dnscrypt-proxy-foreign.service 
```
Test:    

```
$ sudo systemctl restart dnscrypt-proxy
$ sudo systemctl start dnscrypt-proxy-foreign
$ dig google.com @127.0.0.1 -p 5533 +short
172.253.124.139
172.253.124.101
172.253.124.100
172.253.124.102
172.253.124.138
172.253.124.113
$ dig google.com @127.0.0.1 -p 25533 +short
142.250.217.142
```

### 2. dnsmasq flows
Clone the flow control repository:    

```
cd /opt
git clone https://github.com/felixonmars/dnsmasq-china-list
ln -sf /opt/dnsmasq-china-list/accelerated-domains.china.conf  /etc/dnsmasq.d/accelerated-domains.china.conf 
ln -sf /opt/dnsmasq-china-list/google.china.conf /etc/dnsmasq.d/google.china.conf
ln -sf /opt/dnsmasq-china-list/apple.china.conf /etc/dnsmasq.d/apple.china.conf
ln -sf /opt/dnsmasq-china-list/bogus-nxdomain.china.conf /etc/dnsmasq.d/bogus-nxdomain.china.conf 
sed -i 's|114.114.114.114|127.0.0.1#5533|g' accelerated-domains.china.conf
```
Install dnsmasq:    

```
$ sudo apt install -y dnsmasq
$ sudo vim /etc/resolv.conf
nameserver 127.0.0.1
$ sudo chattr +i /etc/resolv.conf
$ cat /etc/dnsmasq.conf
log-queries
log-facility=/var/log/dnsmasq.log
no-hosts
bogus-nxdomain=119.29.29.29
cache-size=1000
port=53
server=127.0.0.1#25533
$ sudo systemctl restart dnsmasq
```
Examine:    

```
$ dig www.google.com
$ curl www.google.com
```
Change the dncrypt-proxy listend socket:    

```
$ vim /lib/systemd/system/dnscrypt-proxy.socket
[Socket]
ListenStream=127.0.2.1:20153
ListenDatagram=127.0.2.1:20153
```
### 3. reference url
`https://dnscrypt.info/public-servers/`
