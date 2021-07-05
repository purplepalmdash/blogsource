+++
title= "WorkingTipsOnLXDAndJuju"
date = "2021-07-03T15:53:23+08:00"
description = "WorkingTipsOnLXDAndJuju"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Before
A LXD cluster and juju status:    

lxd cluster status and lxc instance before deploy work load:    

```
test@freeedge1:~$ lxc cluster list
+-----------+---------------------------+----------+--------------+----------------+-------------+--------+-------------------+
|   NAME    |            URL            | DATABASE | ARCHITECTURE | FAILURE DOMAIN | DESCRIPTION | STATE  |      MESSAGE      |
+-----------+---------------------------+----------+--------------+----------------+-------------+--------+-------------------+
| freeedge1 | https://192.168.89.2:8443 | YES      | x86_64       | default        |             | ONLINE | Fully operational |
+-----------+---------------------------+----------+--------------+----------------+-------------+--------+-------------------+
| freeedge2 | https://192.168.89.3:8443 | YES      | x86_64       | default        |             | ONLINE | Fully operational |
+-----------+---------------------------+----------+--------------+----------------+-------------+--------+-------------------+
| freeedge3 | https://192.168.89.4:8443 | YES      | x86_64       | default        |             | ONLINE | Fully operational |
+-----------+---------------------------+----------+--------------+----------------+-------------+--------+-------------------+
| freeedge4 | https://192.168.89.5:8443 | YES      | x86_64       | default        |             | ONLINE | Fully operational |
+-----------+---------------------------+----------+--------------+----------------+-------------+--------+-------------------+
test@freeedge1:~$ lxc ls
+------+-------+------+------+------+-----------+----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS | LOCATION |
+------+-------+------+------+------+-----------+----------+
```

`juju boostrap` for provisioning a machine with LXD and create a controller
running within it:    

```
$ juju bootstrap localhost overlord
Creating Juju controller "overlord" on localhost/localhost
Looking for packaged Juju agent version 2.9.5 for amd64
WARNING Got error requesting "https://streams.canonical.com/juju/tools/streams/v1/index2.sjson": Get "https://streams.canonical.com/juju/tools/streams/v1/index2.sjson": dial tcp [2001:67c:1360:8001::33]:443: connect: network is unreachable
No packaged binary found, preparing local Juju agent binary
To configure your system to better support LXD containers, please see: https://github.com/lxc/lxd/blob/master/doc/production-setup.md
Launching controller instance(s) on localhost/localhost...
 - juju-5a91a2-0 (arch=amd64)                 
Installing Juju agent on bootstrap instance
Fetching Juju Dashboard 0.7.1
Waiting for address
Attempting to connect to 192.168.89.155:22
Connected to 192.168.89.155
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 192.168.89.155 to verify accessibility...

Bootstrap complete, controller "overlord" is now available
Controller machines are in the "controller" model
Initial model "default" added
```
This command automatically create a new lxd instance and running the script in
it,  like:    

```
$ test@freeedge1:~$ lxc ls
+---------------+---------+-----------------------+------+-----------+-----------+-----------+
|     NAME      |  STATE  |         IPV4          | IPV6 |   TYPE    | SNAPSHOTS | LOCATION  |
+---------------+---------+-----------------------+------+-----------+-----------+-----------+
| juju-5a91a2-0 | RUNNING | 192.168.89.155 (eth0) |      | CONTAINER | 0         | freeedge1 |
+---------------+---------+-----------------------+------+-----------+-----------+-----------+
```

添加juju machines:    

```
test@freeedge1:~$ juju add-machine -n 2
created machine 0
created machine 1
test@freeedge1:~$ juju machines
Machine  State    DNS  Inst id  Series  AZ  Message
0        pending       pending  focal       starting
1        pending       pending  focal       starting
```
等待一段时间，直到状态变为`started`:    

```
$ juju machines
Machine  State    DNS             Inst id        Series  AZ  Message
0        started  192.168.89.197  juju-a5a008-0  focal       Running
1        started  192.168.89.173  juju-a5a008-1  focal       Running
```
实际上是lxc工作负载，exec进入到该实例中可发现其实是没有做任何资源限制的lxc工作实例, 如果资源需要做限制，则可以参考`https://juju.is/docs/olm/constraints#heading--constraints-and-lxd-containers`:    

```
test@freeedge1:~$ lxc ls
+---------------+---------+-----------------------+------+-----------+-----------+-----------+
|     NAME      |  STATE  |         IPV4          | IPV6 |   TYPE    | SNAPSHOTS | LOCATION  |
+---------------+---------+-----------------------+------+-----------+-----------+-----------+
| juju-5a91a2-0 | RUNNING | 192.168.89.155 (eth0) |      | CONTAINER | 0         | freeedge1 |
+---------------+---------+-----------------------+------+-----------+-----------+-----------+
| juju-a5a008-0 | RUNNING | 192.168.89.197 (eth0) |      | CONTAINER | 0         | freeedge2 |
+---------------+---------+-----------------------+------+-----------+-----------+-----------+
| juju-a5a008-1 | RUNNING | 192.168.89.173 (eth0) |      | CONTAINER | 0         | freeedge2 |
+---------------+---------+-----------------------+------+-----------+-----------+-----------+

```
销毁刚才创建的machine:    

```
test@freeedge1:~$ juju remove-machine 0
removing machine 0
test@freeedge1:~$ juju machines
Machine  State    DNS             Inst id        Series  AZ  Message
0        stopped  192.168.89.197  juju-a5a008-0  focal       Running
1        started  192.168.89.173  juju-a5a008-1  focal       Running

test@freeedge1:~$ juju remove-machine 1
removing machine 1
test@freeedge1:~$ juju machines
Machine  State    DNS             Inst id        Series  AZ  Message
1        stopped  192.168.89.173  juju-a5a008-1  focal       Running
```
创建一个`hello-juju`的charmed operator:    

```
test@freeedge1:~$ juju deploy hello-juju
Located charm "hello-juju" in charm-hub, revision 8
Deploying "hello-juju" from charm-hub charm "hello-juju", revision 8 in channel stable
test@freeedge1:~$ juju status
Model    Controller  Cloud/Region         Version  SLA          Timestamp
default  overlord    localhost/localhost  2.9.5    unsupported  16:27:44+08:00

App         Version  Status  Scale  Charm       Store     Channel  Rev  OS      Message
hello-juju           active      1  hello-juju  charmhub  stable     8  ubuntu  

Unit           Workload  Agent  Machine  Public address  Ports   Message
hello-juju/0*  active    idle   2        192.168.89.160  80/tcp  

Machine  State    DNS             Inst id        Series  AZ  Message
2        started  192.168.89.160  juju-a5a008-2  focal       Running

test@freeedge1:~$ juju expose hello-juju
test@freeedge1:~$ juju status
Model    Controller  Cloud/Region         Version  SLA          Timestamp
default  overlord    localhost/localhost  2.9.5    unsupported  16:28:06+08:00

App         Version  Status  Scale  Charm       Store     Channel  Rev  OS      Message
hello-juju           active      1  hello-juju  charmhub  stable     8  ubuntu  

Unit           Workload  Agent  Machine  Public address  Ports   Message
hello-juju/0*  active    idle   2        192.168.89.160  80/tcp  

Machine  State    DNS             Inst id        Series  AZ  Message
2        started  192.168.89.160  juju-a5a008-2  focal       Running
```

![/images/2021_07_03_16_31_31_680x551.jpg](/images/2021_07_03_16_31_31_680x551.jpg)

### 部署charmed Kubernetes
记录一下步骤, 在LXD集群的情况下，直接部署会出现以下错误:    

```
# juju deploy charmed-kubernetes
# juju status
Machine  State  DNS  Inst id  Series  AZ  Message
0        down        pending  focal       Failed creating instance record: Failed initialising instance: Failed loading storage pool: No such object
1        down        pending  focal       Failed creating instance record: Failed initialising instance: Failed loading storage pool: No such object
2        down        pending  focal       Failed creating instance record: Failed initialising instance: Failed loading storage pool: No such object
3        down        pending  focal       Failed creating instance record: Failed initialising instance: Failed loading storage pool: No such object
....
```

这是因为没有默认的`default`格式的storage定义，

```
test@freeedge1:~$ lxc storage list
+-------+--------+-------------+---------+---------+
| NAME  | DRIVER | DESCRIPTION | USED BY |  STATE  |
+-------+--------+-------------+---------+---------+
| local | zfs    |             | 3       | CREATED |
+-------+--------+-------------+---------+---------+
```

先行删除已经部署好的`charmed-kubernetes`（当前没有直接删除charmed
operator的方法，只能将app都删除):    

```
 juju remove-application hello-juju
 juju remove-application containerd
 juju remove-application easyrsa
 juju remove-application etcd
 juju remove-application flannel
 juju remove-application kubeapi-load-balancer
 juju remove-application kubernetes-master
 juju remove-application kubernetes-worker
```
所有节点上创建相同的目录并添加到一个新的storage定义:    

```
 $ ansible -i hosts.ini all -m shell -a "sudo mkdir -p /data/lxd && sudo chmod 777 -R /data/lxd"
$ lxc storage create --target freeedge1 default dir source=/data/lxd 
Storage pool default pending on member freeedge1
$ lxc storage create --target freeedge2 default dir source=/data/lxd 
Storage pool default pending on member freeedge2
$ lxc storage create --target freeedge3 default dir source=/data/lxd 
Storage pool default pending on member freeedge3
$ lxc storage create --target freeedge4 default dir source=/data/lxd 
Storage pool default pending on member freeedge4
$ lxc storage create default dir
Storage pool default created
$ lxc storage volume create default lxdvol --target freeedge1
Storage volume lxdvol created
$ lxc storage volume show default lxdvol --target freeedge1
config: {}
description: ""
name: lxdvol
type: custom
used_by: []
location: freeedge1
content_type: filesystem
```
各种乱七八糟的操作以后，清空:    

```
juju destroy-controller overlord
```
现在重新开始部署juju(参考`https://juju.is/docs/olm/lxd`):    

```
$ juju bootstrap localhost overlord
$ juju deploy charmed-kubernetes
Located bundle "charmed-kubernetes" in charm-hub, revision 679
WARNING "services" key found in bundle file is deprecated, superseded by "applications" key.
Located charm "containerd" in charm-store, revision 130
Located charm "easyrsa" in charm-store, revision 384
Located charm "etcd" in charm-store, revision 594
Located charm "flannel" in charm-store, revision 558
Located charm "kubeapi-load-balancer" in charm-store, revision 798
Located charm "kubernetes-master" in charm-store, revision 1008
Located charm "kubernetes-worker" in charm-store, revision 768
Executing changes:
- upload charm containerd from charm-store for series focal with architecture=amd64
- deploy application containerd from charm-store on focal
- set annotations for containerd
- upload charm easyrsa from charm-store for series focal with architecture=amd64
- deploy application easyrsa from charm-store on focal
  added resource easyrsa
- set annotations for easyrsa
- upload charm etcd from charm-store for series focal with architecture=amd64
- deploy application etcd from charm-store on focal
  added resource core
  added resource etcd
  added resource snapshot
- set annotations for etcd
- upload charm flannel from charm-store for series focal with architecture=amd64
- deploy application flannel from charm-store on focal
  added resource flannel-amd64
  added resource flannel-arm64
  added resource flannel-s390x
- set annotations for flannel
- upload charm kubeapi-load-balancer from charm-store for series focal with architecture=amd64
- deploy application kubeapi-load-balancer from charm-store on focal
- expose all endpoints of kubeapi-load-balancer and allow access from CIDRs 0.0.0.0/0 and ::/0
- set annotations for kubeapi-load-balancer
- upload charm kubernetes-master from charm-store for series focal with architecture=amd64
- deploy application kubernetes-master from charm-store on focal
  added resource cdk-addons
  added resource core
  added resource kube-apiserver
  added resource kube-controller-manager
  added resource kube-proxy
  added resource kube-scheduler
  added resource kubectl
- set annotations for kubernetes-master
- upload charm kubernetes-worker from charm-store for series focal with architecture=amd64
- deploy application kubernetes-worker from charm-store on focal
  added resource cni-amd64
  added resource cni-arm64
  added resource cni-s390x
  added resource core
  added resource kube-proxy
  added resource kubectl
  added resource kubelet
- expose all endpoints of kubernetes-worker and allow access from CIDRs 0.0.0.0/0 and ::/0
- set annotations for kubernetes-worker
- add relation kubernetes-master:kube-api-endpoint - kubeapi-load-balancer:apiserver
- add relation kubernetes-master:loadbalancer - kubeapi-load-balancer:loadbalancer
- add relation kubernetes-master:kube-control - kubernetes-worker:kube-control
- add relation kubernetes-master:certificates - easyrsa:client
- add relation etcd:certificates - easyrsa:client
- add relation kubernetes-master:etcd - etcd:db
- add relation kubernetes-worker:certificates - easyrsa:client
- add relation kubernetes-worker:kube-api-endpoint - kubeapi-load-balancer:website
- add relation kubeapi-load-balancer:certificates - easyrsa:client
- add relation flannel:etcd - etcd:db
- add relation flannel:cni - kubernetes-master:cni
- add relation flannel:cni - kubernetes-worker:cni
- add relation containerd:containerd - kubernetes-worker:container-runtime
- add relation containerd:containerd - kubernetes-master:container-runtime
- add unit easyrsa/0 to new machine 0
- add unit etcd/0 to new machine 1
- add unit etcd/1 to new machine 2
- add unit etcd/2 to new machine 3
- add unit kubeapi-load-balancer/0 to new machine 4
- add unit kubernetes-master/0 to new machine 5
- add unit kubernetes-master/1 to new machine 6
- add unit kubernetes-worker/0 to new machine 7
- add unit kubernetes-worker/1 to new machine 8
- add unit kubernetes-worker/2 to new machine 9
Deploy of bundle completed.
```
部署中可以通过`juju status`看即时的状态变更:    

![/images/2021_07_03_17_51_14_1359x899.jpg](/images/2021_07_03_17_51_14_1359x899.jpg)

涉及到的落地实体:    

![/images/2021_07_03_17_51_58_956x946.jpg](/images/2021_07_03_17_51_58_956x946.jpg)

安装`kubectl`用于管控集群:    

```
test@freeedge1:~$ sudo snap install kubectl --classic
.....
kubectl 1.21.1 from Canonical✓ installed
```
中间状态：    

![/images/2021_07_03_18_00_15_1526x913.jpg](/images/2021_07_03_18_00_15_1526x913.jpg)

![/images/2021_07_03_18_07_12_1525x940.jpg](/images/2021_07_03_18_07_12_1525x940.jpg)

![/images/2021_07_03_18_09_07_1489x922.jpg](/images/2021_07_03_18_09_07_1489x922.jpg)

集群就绪：   

![/images/2021_07_03_18_40_34_1384x934.jpg](/images/2021_07_03_18_40_34_1384x934.jpg)


```
$ juju scp kubernetes-master/0:config ~/.kube/config
$ kubectl get nodes -A
NAME            STATUS   ROLES    AGE   VERSION
juju-08ae08-7   Ready    <none>   19m   v1.21.1
juju-08ae08-8   Ready    <none>   19m   v1.21.1
juju-08ae08-9   Ready    <none>   13m   v1.21.1

```

![/images/2021_07_03_18_42_18_1481x912.jpg](/images/2021_07_03_18_42_18_1481x912.jpg)


添加节点:    

```
$ juju add-unit kubernetes-worker
```
![/images/2021_07_03_18_43_47_709x158.jpg](/images/2021_07_03_18_43_47_709x158.jpg)

增加完后：    

```
$ kubectl get nodes -A                                                                                                                                                      
NAME             STATUS   ROLES    AGE   VERSION
juju-08ae08-10   Ready    <none>   12m   v1.21.1
juju-08ae08-7    Ready    <none>   42m   v1.21.1
juju-08ae08-8    Ready    <none>   42m   v1.21.1
juju-08ae08-9    Ready    <none>   36m   v1.21.1
```
增加3个:     

```
$ juju add-unit kubernetes-worker -n 3
$ juju status
.....
11       pending                  pending         focal       starting
12       pending                  pending         focal       starting
13       pending                  pending         focal       starting
$ kubectl get nodes -A
NAME             STATUS   ROLES    AGE     VERSION
juju-08ae08-10   Ready    <none>   29m     v1.21.1
juju-08ae08-11   Ready    <none>   5m2s    v1.21.1
juju-08ae08-12   Ready    <none>   3m21s   v1.21.1
juju-08ae08-13   Ready    <none>   86s     v1.21.1
juju-08ae08-7    Ready    <none>   59m     v1.21.1
juju-08ae08-8    Ready    <none>   59m     v1.21.1
juju-08ae08-9    Ready    <none>   53m     v1.21.1

```

