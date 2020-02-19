+++
title = "kubespray2.12.0离线化手记"
date = "2019-12-19T16:34:09+08:00"
description = "onkubespray2.12.0"
keywords = ["Linux"]
categories = ["Technology"]
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
# docker pull xueshanf/install-socat:latest
# docker images | sed -n '1!p' | awk {'print $1":"$2'} | tr '\n' ' '
nginx:1.17 gcr.io/google-containers/k8s-dns-node-cache:1.15.8 gcr.io/google-containers/kube-proxy:v1.16.3 gcr.io/google-containers/kube-apiserver:v1.16.3 gcr.io/google-containers/kube-controller-manager:v1.16.3 gcr.io/google-containers/kube-scheduler:v1.16.3 lachlanevenson/k8s-helm:v2.16.1 gcr.io/kubernetes-helm/tiller:v2.16.1 coredns/coredns:1.6.0 calico/node:v3.7.3 calico/cni:v3.7.3 calico/kube-controllers:v3.7.3 gcr.io/google_containers/metrics-server-amd64:v0.3.3 gcr.io/google-containers/cluster-proportional-autoscaler-amd64:1.6.0 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1 quay.io/coreos/etcd:v3.3.10 gcr.io/google-containers/addon-resizer:1.8.3 gcr.io/google-containers/pause:3.1 gcr.io/google_containers/pause-amd64:3.1 xueshanf/install-socat:latest
# docker save -o k8simages.tar nginx:1.17 gcr.io/google-containers/k8s-dns-node-cache:1.15.8 gcr.io/google-containers/kube-proxy:v1.16.3 gcr.io/google-containers/kube-apiserver:v1.16.3 gcr.io/google-containers/kube-controller-manager:v1.16.3 gcr.io/google-containers/kube-scheduler:v1.16.3 lachlanevenson/k8s-helm:v2.16.1 gcr.io/kubernetes-helm/tiller:v2.16.1 coredns/coredns:1.6.0 calico/node:v3.7.3 calico/cni:v3.7.3 calico/kube-controllers:v3.7.3 gcr.io/google_containers/metrics-server-amd64:v0.3.3 gcr.io/google-containers/cluster-proportional-autoscaler-amd64:1.6.0 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1 quay.io/coreos/etcd:v3.3.10 gcr.io/google-containers/addon-resizer:1.8.3 gcr.io/google-containers/pause:3.1 gcr.io/google_containers/pause-amd64:3.1 xueshanf/install-socat:latest; xz -T4 k8simages.tar
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
### More pkgs
Use the old deb repository for installing ansible:     

```
$ cp old_1804debs.tar.xz ~/YourWebServer
$ tar xJvf old_1804debs.tar.xz
$ sudo vim /etc/apt/sources.list
deb [trusted=yes]  http://192.168.122.1/ansible_bionic ./
$ sudo apt-get update -y && sudo DEBIAN_FRONTEND=noninteractive apt-get install -y ansible python-netaddr
```

more pkgs should be installed manually and copy to `/root/debs`:    

```
# apt-get install -y iputils-ping nethogs python-netaddr build-essential bind9 bind9utils nfs-common nfs-kernel-server ntpdate ntp tcpdump iotop unzip wget apt-transport-https socat rpcbind arping fping python-apt ipset ipvsadm pigz nginx docker-registry
# cd /root/debs
# wget http://209.141.35.192/netdata_1.18.1_amd64_bionic.deb
# apt-get install  ./netdata_1.18.1_amd64_bionic.deb
# find /var/cache | grep deb$ | xargs -I % cp % ./
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
```
### Offline registry setup
On a running secureregistry server do following:    

```
# systemctl stop secureregistryserver
# cd /opt/local/secureregistryserver/
# mv data data.back
# docker-compose up
# docker push xxxxx
```
Your docker push item is listed as(v1.16.3):     

```
docker push gcr.io/google-containers/k8s-dns-node-cache:1.15.8
docker push gcr.io/google-containers/kube-proxy:v1.16.3
docker push gcr.io/google-containers/kube-apiserver:v1.16.3
docker push gcr.io/google-containers/kube-controller-manager:v1.16.3
docker push gcr.io/google-containers/kube-scheduler:v1.16.3
docker push lachlanevenson/k8s-helm:v2.16.1
docker push gcr.io/kubernetes-helm/tiller:v2.16.1
docker push coredns/coredns:1.6.0
docker push calico/node:v3.7.3
docker push calico/cni:v3.7.3
docker push calico/kube-controllers:v3.7.3
docker push gcr.io/google_containers/metrics-server-amd64:v0.3.3
docker push gcr.io/google-containers/cluster-proportional-autoscaler-amd64:1.6.0
docker push gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1
docker push quay.io/coreos/etcd:v3.3.10
docker push gcr.io/google-containers/addon-resizer:1.8.3
docker push gcr.io/google-containers/pause:3.1
docker push gcr.io/google_containers/pause-amd64:3.1
docker push xueshanf/install-socat:latest
docker push nginx:1.17
```
tar docker.tar.gz:    

```
# cd /opt/local/secureregistryserver/data
# tar czvf docker.tar.gz docker/

```
### Upgrade
From v1.15.3 to v1.16.3, steps:    

```
$ pwd
0_preinstall/roles/kube-deploy/files
$ ls
1604debs.tar.xz  1804debs.tar.xz  calicoctl-linux-amd64  cni-plugins-linux-amd64-v0.8.1.tgz  dns  docker-compose  docker.tar.gz  dockerDebs.tar.gz  gpg  hyperkube  kubeadm  nginx  ntp.conf
```
Generate 1804debs.tar.xz and replace:    

```
# cp -r /root/debs ./Rong
# tar cJvf 1804debs.tar.xz Rong
```
Calculate calicoctl/ , it's the same md5, so needn't replacement.    

`docker.tar.gz` should be replaced with the newer one.     

Docker version upgradeed to 19.03.5, so we need to replace the old ones.   

```
# tar xzvf dockerDebs.tar.gz  -C tmp/
ubuntu/dists/bionic/pool/stable/amd64/containerd.io_1.2.10-2_amd64.deb
ubuntu/dists/bionic/pool/stable/amd64/docker-ce-cli_19.03.3~3-0~ubuntu-bionic_amd64.deb
ubuntu/dists/bionic/pool/stable/amd64/docker-ce_18.09.7~3-0~ubuntu-bionic_amd64.deb
``` 
apt-mirror for syncing on internet:    

```
$ sudo vim /etc/apt/mirror.list
set base_path    /media/sda/tmp/apt-mirror
set nthreads     20
set _tilde 0
deb https://download.docker.com/linux/ubuntu bionic stable
deb https://download.docker.com/linux/ubuntu xenial stable
$ sudo apt-mirror

```
Too slow for the fucking gfw!!!    

After apt-mirror, we have to rsync using following command:     

```
$ pwd
/media/sda/tmp/apt-mirror/mirror/download.docker.com/linux/ubuntu
$ ls
dists
$ rsync -a -e 'ssh -p 2345 ' --progress dists/ root@192.168.111.11:/destination/ubuntu/dists/
```

wget the gpg file:    

```
$ wget https://download.docker.com/linux/ubuntu/gpg
$ tar czvf dockerDebs.tar.gz gpg ubuntu/
$ ls -l -h dockerDebs.tar.gz
-rw-r--r-- 1 root root 144M Dec 23 17:41 dockerDebs.tar.gz
$ cp dockerDebs.tar.gz ~/0_preinstall/roles/kube-deploy/files
```
Binary replacement:    

```
previsous:    
 hyperkube  kubeadm  
current:    
kubeadm-v1.16.3-amd64 kubectl-v1.16.3-amd64 kubelet-v1.16.3-amd64
```
Edit the file, since in v1.16.3 we didn't use hyperkube:    

```
$ vim deploy-ubuntu/tasks/main.yml
  - name: "upload static files to /usr/local/static"
    copy:
      src: "{{ item }}"
      dest: /usr/local/static/
      owner: root
      group: root
      mode: 0777
    with_items:
      #- files/hyperkube
      - files/calicoctl-linux-amd64
      - files/kubeadm-v1.16.3-amd64
      - files/kubectl-v1.16.3-amd64
      - files/kubelet-v1.16.3-amd64
      #- files/kubeadm
      - files/cni-plugins-linux-amd64-v0.8.1.tgz
      #- files/dockerDebs.tar.gz
      - files/gpg
```
Add sysctl items:    

```
# vim ./roles/kubernetes/preinstall/tasks/0080-system-configurations.yml
- name: set fs inotify.max_user_watches to 1048576
  sysctl:
    sysctl_file: "{{ sysctl_file_path }}"
    name: fs.inotify.max_user_watches
    value: 1048576
    state: present
    reload: yes
```
Added some files like `./roles/kubernetes/preinstall/tasks/0000-xxx-ubuntu.yml`, minimum modifications to kubespray source code, you can use bcompare for viewing.      
