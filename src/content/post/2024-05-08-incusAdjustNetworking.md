+++
title= "incusAdjustNetworking"
date = "2024-05-08T08:22:25+08:00"
description = "incusAdjustNetworking"
keywords = ["Technology"]
categories = ["Technology"]
+++
The default networking bridge `incusbr0` enabled the dhcp by default, that's not good for using dhcpd service in containers, so I have to remove the default behavior of the `incusbr0`, and add a new behavior for it.    

Directly delete the bridge will get an error:   

```
$ incus network delete incusbr0
Error: The network is currently in use
```
Show this network's usage:    

```
$ incus network show incusbr0
config:
  ipv4.address: 10.147.148.1/24
  ipv4.nat: "true"
  ipv6.address: none
description: ""
name: incusbr0
type: bridge
used_by:
- /1.0/instances/fogincuschinese
- /1.0/instances/foginlxc
- /1.0/profiles/default
managed: true
status: Created
locations:
- none
```
Edit its profile:    

```
$ incus profile edit default
config: {}
description: Default Incus profile
devices:
-  eth0:
-    name: eth0
-    network: incusbr0
-    type: nic
  root:
    path: /
    pool: default
    type: disk
name: default
used_by:
- /1.0/instances/foginlxc
- /1.0/instances/fogincuschinese
```
Now you could delete this networking via:    

```
$ incus network delete incusbr0
Network incusbr0 deleted
```
RE-create the networking via following command(dhcpv4/v6 disabled):    

```
$ incus network create incusbr0 ipv4.dhcp=false ipv6.dhcp=false ipv4.address=10.147.148.1/24
Network incusbr0 created
```
Check this networking:    

```
$ ip a show incusbr0
10: incusbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 00:16:3e:c9:c4:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.147.148.1/24 scope global incusbr0
       valid_lft forever preferred_lft forever
    inet6 fd42:1515:fb8e:9dab::1/64 scope global 
       valid_lft forever preferred_lft forever
```
RE-Add the networking profile into default:    

```
$ incus profile edit default
...
description: Default Incus profile
devices:
+  eth0:
+    name: eth0
+    network: incusbr0
+    type: nic
  root:
...

```
Re-lauch the previously stopped container instance:    

```
$ incus start fogincuschinese
$ incus list
+-----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
|      NAME       |  STATE  |         IPV4          |                     IPV6                      |   TYPE    | SNAPSHOTS |
+-----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| fogincuschinese | RUNNING | 10.147.148.100 (eth0) | fd42:1515:fb8e:9dab:216:3eff:fef3:8307 (eth0) | CONTAINER | 0         |
+-----------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
```
Until now you could enable the dhcpd in container and then use forwarding rules for redirect to host.    

Final command:      

```
incus network create incusbr0 ipv4.dhcp=false ipv4.address=10.147.148.1/24 ipv4.nat=true ipv6.address=none
```
