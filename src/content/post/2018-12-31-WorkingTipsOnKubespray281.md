+++
title = "WorkingTipsOnKubespray281"
date = "2018-12-31T11:20:06+08:00"
description = "Kubespray281"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Changes
#### 1. download items
In `kubespray-2.8.1/roles/download/defaults/main.yml`, get download info from
following definition:    

```
kubeadm_download_url: "https://storage.googleapis.com/kubernetes-release/release/{{ kubead
m_version }}/bin/linux/{{ image_arch }}/kubeadm"
hyperkube_download_url: "https://storage.googleapis.com/kubernetes-release/release/{{ kube
_version }}/bin/linux/amd64/hyperkube"
cni_download_url: "https://github.com/containernetworking/plugins/releases/download/{{ cni
_version }}/cni-plugins-{{ image_arch }}-{{ cni_version }}.tgz"
```
The `cni_version` is defined in  following file:    

```
./roles/download/defaults/main.yml:cni_version: "v0.6.0"
```

Download from following position:    

```
https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/kubeadm
https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/hyperkube
https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz
```

Changes to:    

```
#kubeadm_download_url: "https://storage.googleapis.com/kubernetes-release/release/{{ kubeadm_version }}/bin/linux/{{ image_arch }}/kubeadm"
#hyperkube_download_url: "https://storage.googleapis.com/kubernetes-release/release/{{ kube_version }}/bin/linux/amd64/hyperkube"
etcd_download_url: "https://github.com/coreos/etcd/releases/download/{{ etcd_version }}/etcd-{{ etcd_version }}-linux-amd64.tar.gz"
#cni_download_url: "https://github.com/containernetworking/plugins/releases/download/{{ cni_version }}/cni-plugins-{{ image_arch }}-{{ cni_version }}.tgz"
kubeadm_download_url: "http://portus.xxxx.com:8888/kubeadm"
hyperkube_download_url: "http://portus.xxxx.com:8888/hyperkube"
cni_download_url: "http://portus.xxxx.com:8888/cni-plugins-{{ image_arch }}-{{ cni_version }}.tgz"
```
#### 2. dashboard
`kubespray-2.8.1/roles/kubernetes-apps/ansible/templates/dashboard.yml.j2`, add NodePort definition:     

```
spec:
+  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
```
#### 3. bootstrap-os
Added in files:    

```
portus.crt
server.crt
ntp.conf
```

`kubespray-2.8.1/roles/bootstrap-os/tasks/bootstrap-ubuntu.yml`, modify according to previous version.     

#### 4. kube-deploy
TBD, changes later

#### 5. reset
`kubespray-2.8.1/roles/reset/tasks/main.yml`

```
    - /etc/cni
    - "{{ nginx_config_dir }}"
#    - /etc/dnsmasq.d
#    - /etc/dnsmasq.conf
#    - /etc/dnsmasq.d-available
```

#### 6. inventory definition
`/kubespray-2.8.1/inventory/sample/group_vars/k8s-cluster/addons.yml` 

``` 
enable helm and metric-server
```

Edit `kubespray-2.8.1/inventory/sample/group_vars/k8s-cluster/k8s-cluster.yml` file:    

```
helm_stable_repo_url: "https://portus.xxxx.com:5000/chartrepo/kubesprayns"

also notice the version of kubeadm, for example v1.12.4
```
remove the hosts.ini file.   

#### 7. kubeadm images
Use an official vagrant definition for downloading kubeadm images. 

### Vagrant temp
Vagrant create temp machines.   

Stop the service:    

```
sudo systemctl stop secureregistryserver.service
```
Remove the old registry data, and start a new instance

```
sudo rm -rf /usr/local/secureregistryserver/data/*
sudo systemcel start secureregistryserver.service
```
Load:    

```
scp ./all.tar.bz2 vagrant@172.17.129.101:/home/vagrant
sudo docker load<all.tar.bz2
```
Then docker push all of the loaded images, compress the folder:    

```
sudo systemcel stop secureregistryserver.service
tar cvf /usr/local/secureregistryserver/
xz /usr/local/secureregistryserver.tar
```
With the tar.xz, contains all of the offline images.    
