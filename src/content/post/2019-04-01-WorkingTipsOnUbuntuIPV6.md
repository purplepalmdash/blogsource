+++
title = "WorkingTipsOnUbutunIPV6"
date = "2019-04-01T09:01:00+08:00"
description = "WorkingTipsOnUbutunIPV6"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Conoha ipv6
Find the ipv6 addresses(should have 17 ipv6 addresses together):    

![/images/2019_04_01_09_01_58_1054x360.jpg](/images/2019_04_01_09_01_58_1054x360.jpg)

### Network Configuration
Edit the networking via:     

```
#  vim /etc/network/interfaces
# The primary network interface
auto ens3
iface ens3 inet dhcp
iface ens3 inet6 dhcp 
accept_ra 1

# IPv6 configuration
iface ens3 inet6 static
  pre-up modprobe ipv6
  address xxxxxx
  netmask 64
```
Restart the networking via `service networking restart`, then you will get the
ipv6 address via:     

```
# ifconfig | grep inet6
```

### shadowsocks configuration
Edit the configuration :    

```
# cat shadowsocks.json
{
"server":"::",

.....

"prefer_ipv6": true
}

```
Now restart the service of sss

