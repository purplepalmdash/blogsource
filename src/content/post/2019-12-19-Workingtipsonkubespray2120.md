+++
title = "kubespray2.12.0离线化手记"
date = "2019-12-19T16:34:09+08:00"
description = "onkubespray2.12.0"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
Download the source code:    

```
# wget https://github.com/kubernetes-sigs/kubespray/archive/v2.12.0.tar.gz
```
Install ansible via old Rong/.      

```
$ scp -r Rong test@192.168.121.104:/home/test/
$ cd ~/Rong
$ sudo mv /etc/apt/sources.list /home/test/
$ sudo ./bootstrap.sh
$ sudo mv /home/test/sources.list /etc/apt/
```
Change options:    

```
$ cd ~/kubespray-2.12.0
$ cp ../deploy.key .
$ ssh -i deploy.key root@192.168.121.104
$ exit
$ cp -rfp inventory/sample/ inventory/rong
$ vim inventory/rong/hosts.ini
[all]
kubespray ansible_host=192.168.121.104 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key  ip=192.168.121.104

[kube-master]
kubespray

[etcd]
kubespray

[kube-node]
kubespray

[k8s-cluster:children]
kube-master
kube-node
```
Add some configuration:    

```
$ vim group_vars/k8s-cluster/addons.yml 
dashboard_enabled: true
helm_enabled: true
metrics_server_enabled: true
```
### Speedup
cross the gfw, host machine side:     

```
$ sudo iptables -t nat -A PREROUTING -p tcp -s 192.168.121.0/24 -j DNAT --to-destination 127.0.0.1:12345
$ sudo sysctl -w net.ipv4.conf.all.route_localnet=1
```
vm side:    

```
$ sudo vim /etc/resolv.conf
nameserver 223.5.5.5
nameserver 8.8.8.8
```

### Setup Cluster
Via:    

```
$ ansible-playbook -i inventory/rong/hosts.ini cluster.yml
```

### Fetch things
Get all of the images:    

```
# docker images | sed -n '1!p' | awk {'print $1":"$2'} | tr '\n' ' '
nginx:1.17 gcr.io/google-containers/k8s-dns-node-cache:1.15.8 gcr.io/google-containers/kube-proxy:v1.16.3 gcr.io/google-containers/kube-apiserver:v1.16.3 gcr.io/google-containers/kube-controller-manager:v1.16.3 gcr.io/google-containers/kube-scheduler:v1.16.3 lachlanevenson/k8s-helm:v2.16.1 gcr.io/kubernetes-helm/tiller:v2.16.1 coredns/coredns:1.6.0 calico/node:v3.7.3 calico/cni:v3.7.3 calico/kube-controllers:v3.7.3 gcr.io/google_containers/metrics-server-amd64:v0.3.3 gcr.io/google-containers/cluster-proportional-autoscaler-amd64:1.6.0 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1 quay.io/coreos/etcd:v3.3.10 gcr.io/google-containers/addon-resizer:1.8.3 gcr.io/google-containers/pause:3.1 gcr.io/google_containers/pause-amd64:3.1
# docker save -o k8simages.tar nginx:1.17 gcr.io/google-containers/k8s-dns-node-cache:1.15.8 gcr.io/google-containers/kube-proxy:v1.16.3 gcr.io/google-containers/kube-apiserver:v1.16.3 gcr.io/google-containers/kube-controller-manager:v1.16.3 gcr.io/google-containers/kube-scheduler:v1.16.3 lachlanevenson/k8s-helm:v2.16.1 gcr.io/kubernetes-helm/tiller:v2.16.1 coredns/coredns:1.6.0 calico/node:v3.7.3 calico/cni:v3.7.3 calico/kube-controllers:v3.7.3 gcr.io/google_containers/metrics-server-amd64:v0.3.3 gcr.io/google-containers/cluster-proportional-autoscaler-amd64:1.6.0 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1 quay.io/coreos/etcd:v3.3.10 gcr.io/google-containers/addon-resizer:1.8.3 gcr.io/google-containers/pause:3.1 gcr.io/google_containers/pause-amd64:3.1; xz -T4 k8simages.tar
```
Get debs:    

```
# mkdir /home/test/debs
# find . | grep deb$ | xargs -I % cp % /home/test/debs/
```
Get temp files:    

```
# ls /tmp/releases/
calicoctl                           images/                             kubectl-v1.16.3-amd64               
cni-plugins-linux-amd64-v0.8.1.tgz  kubeadm-v1.16.3-amd64               kubelet-v1.16.3-amd64     
# cp /tmp/releases/* /home/test/file/
```
### Offline
How to do offline installation, to be continued. 

