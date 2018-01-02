+++
title = "UsingLocalRdesktopForAcrossingSomething"
date = "2017-07-31T12:23:57+08:00"
description = ""
keywords = ["Linux"]
categories = ["Linux"]
+++
### MAC Address Spoofing
First you have to cheat your remote machine via changing your own MAC address
from the origin one to the remote box address.     

There are many methods in:   
[https://wiki.archlinux.org/index.php/MAC_address_spoofing](https://wiki.archlinux.org/index.php/MAC_address_spoofing)    

My method is via changing the systemd-networkd:   

```
$ pwd
/etc/systemd/network
$ cat 00-default.link 
[Match]
MACAddress=xx:xx:xx:xx:xx

[Link]
MACAddress=xx:xx:xx:xx:xx
NamePolicy=kernel database onboard slot path
```
After your changing, reboot your system.    

### Iptables Changing
Add following lines into my own iptables rules:    

```
sudo iptables -A OUTPUT -o br0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A OUTPUT -o br0 -p tcp --dport 3389 -j ACCEPT
sudo iptables -A OUTPUT -o br0 -j DROP
sudo iptables -A OUTPUT -o enp0s25 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A OUTPUT -o enp0s25 -p tcp --dport 3389 -j ACCEPT
sudo iptables -A OUTPUT -o enp0s25 -j DROP
sudo iptables -A INPUT -i br0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i br0 -p tcp --dport 3389 -j ACCEPT
sudo iptables -A INPUT -i br0 -j DROP
sudo iptables -A INPUT -i enp0s25 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i enp0s25 -p tcp --dport 3389 -j ACCEPT
sudo iptables -A INPUT -i enp0s25 -j DROP
```
Now use the `rdesktop` for viewing the remote desktop, I could get in touch
with the remote machine desktop, now I won't changing the screen for viewing
the remote machine, saving many times.    

### Iptables Save Permenant
Edit the service definition files:    

```
# vim /etc/iptables/iptables.rules
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [251:34691]
-A OUTPUT -o br0 -m state --state ESTABLISHED,RELATED -j ACCEPT
-A OUTPUT -o br0 -p tcp --dport 3389 -j ACCEPT
-A OUTPUT -o br0 -j DROP
-A OUTPUT -o enp0s25 -m state --state ESTABLISHED,RELATED -j ACCEPT
-A OUTPUT -o enp0s25 -p tcp --dport 3389 -j ACCEPT
-A OUTPUT -o enp0s25 -j DROP
-A INPUT -i br0 -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -i br0 -p tcp --dport 3389 -j ACCEPT
-A INPUT -i br0 -j DROP
-A INPUT -i enp0s25 -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -i enp0s25 -p tcp --dport 3389 -j ACCEPT
-A INPUT -i enp0s25 -j DROP
COMMIT
```
Enable the service :    

```
# sudo systemctl enable iptables.service
```

This method won't work properly, because libvirtd also add some rules. 

Finally I have to add the scripts in my awesome startup scripts.    


### Iptables Recovery
Recover the default iptables rules via:   

```
sudo iptables -D OUTPUT -o br0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -D OUTPUT -o br0 -p tcp --dport 3389 -j ACCEPT
sudo iptables -D OUTPUT -o br0 -j DROP
sudo iptables -D OUTPUT -o enp0s25 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -D OUTPUT -o enp0s25 -p tcp --dport 3389 -j ACCEPT
sudo iptables -D OUTPUT -o enp0s25 -j DROP
sudo iptables -D INPUT -i br0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -D INPUT -i br0 -p tcp --dport 3389 -j ACCEPT
sudo iptables -D INPUT -i br0 -j DROP
sudo iptables -D INPUT -i enp0s25 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -D INPUT -i enp0s25 -p tcp --dport 3389 -j ACCEPT
sudo iptables -D INPUT -i enp0s25 -j DROP
```

