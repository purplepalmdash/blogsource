+++
title = "SoftRouterSetup"
date = "2018-08-20T11:43:35+08:00"
description = "SoftRouterSetup"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 目的
设置内网的独立实验网段，需要一个软路由，做转发。

### 准备
Debian 9.3.0 ISO.     
虚拟机，1核, 512M, debian系统安装, 20G硬盘。   
最小化安装 Debian x86_64系统。    
双网卡，一个连接bridged网络，另一个连接本机上的192.168.122.0/24网络，该网络为虚拟机的默认网络，可通过NAT转换到外头。    

### 配置
安装必要的包:    

```
# apt-get update
# apt-get install net-tools isc-dhcp-server
```
配置网络:    

```
# vim /etc//network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens3
iface ens3 inet static
address 192.168.122.254
netmask 255.255.255.0
gateway 192.168.122.1

auto ens4
iface ens4 inet static
address 10.33.34.1
netmask 255.255.255.0
``` 
配置dhcpd服务器:    

```
# vim /etc/dhcp/dhcpd.conf 
    # dhcpd.conf
    #
    # Sample configuration file for ISC dhcpd
    #
    
    # option definitions common to all supported networks...
    option domain-name "example.org";
    option domain-name-servers ns1.example.org, ns2.example.org;
    
    default-lease-time 600;
    max-lease-time 7200;
    
    # The ddns-updates-style parameter controls whether or not the server will
    # attempt to do a DNS update when a lease is confirmed. We default to the
    # behavior of the version 2 packages ('none', since DHCP v2 didn't
    # have support for DDNS.)
    ddns-update-style none;
    
    # If this DHCP server is the official DHCP server for the local
    # network, the authoritative directive should be uncommented.
    #authoritative;
    
    # Use this to send dhcp log messages to a different log file (you also
    # have to hack syslog.conf to complete the redirection).
    #log-facility local7;
    
    # No service will be given on this subnet, but declaring it helps the 
    # DHCP server to understand the network topology.
    
    #subnet 10.152.187.0 netmask 255.255.255.0 {
    #}
    
    # This is a very basic subnet declaration.
    
    #subnet 10.254.239.0 netmask 255.255.255.224 {
    #  range 10.254.239.10 10.254.239.20;
    #  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
    #}
    
    # This declaration allows BOOTP clients to get dynamic addresses,
    # which we don't really recommend.
    
    #subnet 10.254.239.32 netmask 255.255.255.224 {
    #  range dynamic-bootp 10.254.239.40 10.254.239.60;
    #  option broadcast-address 10.254.239.31;
    #  option routers rtr-239-32-1.example.org;
    #}
    
    # A slightly different configuration for an internal subnet.
    #subnet 10.5.5.0 netmask 255.255.255.224 {
    #  range 10.5.5.26 10.5.5.30;
    #  option domain-name-servers ns1.internal.example.org;
    #  option domain-name "internal.example.org";
    #  option routers 10.5.5.1;
    #  option broadcast-address 10.5.5.31;
    #  default-lease-time 600;
    #  max-lease-time 7200;
    #}
    
    # Hosts which require special configuration options can be listed in
    # host statements.   If no address is specified, the address will be
    # allocated dynamically (if possible), but the host-specific information
    # will still come from the host declaration.
    
    #host passacaglia {
    #  hardware ethernet 0:0:c0:5d:bd:95;
    #  filename "vmunix.passacaglia";
    #  server-name "toccata.example.com";
    #}
    
    # Fixed IP addresses can also be specified for hosts.   These addresses
    # should not also be listed as being available for dynamic assignment.
    # Hosts for which fixed IP addresses have been specified can boot using
    # BOOTP or DHCP.   Hosts for which no fixed address is specified can only
    # be booted with DHCP, unless there is an address range on the subnet
    # to which a BOOTP client is connected which has the dynamic-bootp flag
    # set.
    #host fantasia {
    #  hardware ethernet 08:00:07:26:c0:a5;
    #  fixed-address fantasia.example.com;
    #}
    
    # You can declare a class of clients and then do address allocation
    # based on that.   The example below shows a case where all clients
    # in a certain class get addresses on the 10.17.224/24 subnet, and all
    # other clients get addresses on the 10.0.29/24 subnet.
    
    #class "foo" {
    #  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
    #}
    
    #shared-network 224-29 {
    #  subnet 10.17.224.0 netmask 255.255.255.0 {
    #    option routers rtr-224.example.org;
    #  }
    #  subnet 10.0.29.0 netmask 255.255.255.0 {
    #    option routers rtr-29.example.org;
    #  }
    #  pool {
    #    allow members of "foo";
    #    range 10.17.224.10 10.17.224.250;
    #  }
    #  pool {
    #    deny members of "foo";
    #    range 10.0.29.10 10.0.29.230;
    #  }
    #}
    class "kvm" {
        match if binary-to-ascii(16,8,":",substring(hardware, 1, 2)) = "52:54";
    }
    
    subnet 10.33.34.0	netmask 255.255.255.0 {
    option routers 10.33.34.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 10.33.34.255;
    option domain-name-servers 10.33.34.1;
    option time-offset 0;
    pool {
        range 10.33.34.100	10.33.34.200;
        allow members of "kvm";
    }
    default-lease-time	1209600;
    max-lease-time 1814400;
    }
# vim /etc/default/isc-dhcp-server 
    # Defaults for isc-dhcp-server (sourced by /etc/init.d/isc-dhcp-server)
    
    # Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
    #DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
    #DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf
    
    # Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
    #DHCPDv4_PID=/var/run/dhcpd.pid
    #DHCPDv6_PID=/var/run/dhcpd6.pid
    
    # Additional options to start dhcpd with.
    #	Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
    #OPTIONS=""
    
    # On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
    #	Separate multiple interfaces with spaces, e.g. "eth0 eth1".
    INTERFACESv4="ens4"
    INTERFACESv6=""
```
现在重启服务:    

```
# systemctl restart isc-dhcp-server
```
重启完毕后，所有bridged的虚拟机将得到同样的地址段的地址。    

### 转发
转发到某网段,更改地址ens3为该网段(192.192.189.109)，然后:    

```
# iptables -t nat -A POSTROUTING -s 10.33.34.0/24 -j SNAT --to-source 192.192.189.109
```
开启转发:   

```
# vim /etc/sysctl.conf
net.ipv4.ip_forward=1
```
安装iptables-persistent:    

```
# apt-get install iptables-persistent
```
这样就打通了两个网段之间的联系。  

### 访问网段
访问该网段，Linux下添加:    

```
# route add -net 10.33.34.0/24 gw 192.192.189.109
```

Windows:    

```
# route add 10.33.34.1 mask 255.255.255.0 192.192.189.109
```
But in windows it didn't work.   
