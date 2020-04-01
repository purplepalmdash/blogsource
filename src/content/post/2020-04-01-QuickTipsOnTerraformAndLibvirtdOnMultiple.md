+++
categories = ["Technology"]
date = "2020-04-01T17:27:43+08:00"
description = "QuickTipsOnTerraformAndLibvirtdOnMultiple"
keywords = ["Technology"]
title= "QuickTipsOnTerraformAndLibvirtdOnMultiple"

+++
### terraform configuration
On archlinux, it should be configured as following:      

```
# sudo pacman -S terraform
$ yaourt terraform-libvirt
```
manually create the folder and copy some plugins into the folder:     

```
$ mkdir -p ~/.terraform.d/plugins
$ cp xxxx ~/.terraform.d/plugins
$ ls ~/.terraform.d/plugins
terraform-provider-ansible
```
### libvirtd configuration(Ubuntu)
qemu configuration(Or terraform will complain priviledge):    

```
# vim /etc/libvirt/qemu.conf
security_driver = "none"
```

libvirtd configuration:     

```
$ vim /etc/default/libvirtd
libvirtd_opts="-l"
$ vim /etc/libvirt/libvirtd.conf
listen_tls = 0
listen_tcp = 1
tcp_port = "16509"
listen_addr = "0.0.0.0"
auth_tcp = "none"
```
If you want to use bridge networking, make sure the following configuration is in
sysctl:      

```
# cat >> /etc/sysctl.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
EOF
# sysctl -p /etc/sysctl.conf
```

### dnsmasq dhcpd configuration
In order to configure the dhcpd in bridged networking , we have to configure the
dnsmasq server on gateway machine:      

```
# vim /etc/dnsmasq.conf
bind-interfaces
dhcp-range=10.137.149.100,10.137.149.200,12h
dhcp-option=3,10.137.149.1
dhcp-authoritative
interface=enp3s0
# systemctl restart dnsmasq
```

### terraform networking
Configure network parameter:      

```
# vim main.tf
  network_interface {
		#network_name   = "default"
		bridge = "br0"
    hostname   = "${var.VM_HOSTNAME}-${count.index + 1}"
    wait_for_lease = true
  }
# terraform apply
```
