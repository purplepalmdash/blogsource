+++
title = "KubeSprayISODeployment"
date = "2018-10-17T09:22:15+08:00"
description = "KubeSprayISODeployment"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 文档目的
详细说明使用ISO离线构建Kubernetes集群的步骤。  

### 准备
硬件：至少两台机器（物理机/虚拟机均可），其中一台用作部署服务器，推荐为2核CPU/3G内存以上配置，另一台用作Kubernetes节点，推荐为4核/8G内存以上配置。如果有更多的机器，则可以考虑都作为Kubernetes工作节点接入集群。   
软件: 1604_deploy.iso(部署用ISO)， 1604_node.iso(工作节点用ISO)
### 系统初始化安装
#### 部署节点
插入1604_deploy.iso, 光盘启动系统, 选择English，进入到下一步:   
![/images/2018_10_17_09_28_14_630x472.jpg](/images/2018_10_17_09_28_14_630x472.jpg)

选择`Install Ubuntu Server`:    
![/images/2018_10_17_09_28_35_369x288.jpg](/images/2018_10_17_09_28_35_369x288.jpg)

一路回车，选择默认配置，进入到系统安装, 安装完系统将自动重启。    

系统重启完毕后，按Alt+F2，进入到命令行终端(用户名/密码为root/thinker@1):   

![/images/2018_10_17_09_37_11_460x168.jpg](/images/2018_10_17_09_37_11_460x168.jpg)

配置IP地址:    

```
# vim /etc/network/interfaces
auto ens
- iface ens3 inet dhcp
+ iface ens3 inet static
+ address 192.168.122.178
+ netmask 255.255.255.0
+ gateway 192.168.122.1
+ dns-nameservers 192.168.122.1 192.168.122.178
# vim /etc/ssh/sshd_config
- PermitRootLogin prohibit-password
+ PermitRootLogin yes
# reboot
```
重新启动后，登入命令行，执行:    

```
# ./initial.sh
```
待脚本执行完毕后，部署节点部署完毕。   


#### 工作节点
插入1604_node.iso, 光盘启动系统，选择English， 进入到下一步:    

![/images/2018_10_17_09_30_52_383x356.jpg](/images/2018_10_17_09_30_52_383x356.jpg)

选择`Install XXXX Ubuntu Work Node(Auto-part)`， 进入到下一步,
系统将自动安装，全程不需要人为干预:    

安装完毕后，进入到命令行接口，配置IP地址:    

```
# vim /etc/network/interfaces
auto ens3
iface ens3 inet static
address 192.168.122.179
netmask 255.255.255.0
network 192.168.122.0
broadcast 192.168.122.255
gateway 192.168.122.1
dns-nameservers 192.168.122.1 192.168.122.178
# vim /etc/hostname
node1
# reboot
```
重启完毕后，工作节点准备就绪, 此时运行一下`apt-get
update`更新一下包缓存，否则部署Kubernetes时会报错.   
### 部署Kubernetes
登入部署节点(192.168.122.178), 执行以下步骤:    

```
# cd ansible/kubespray
# vim inventory/test/hosts.ini
```
这里是我们用于配置Kubernetes节点的定义文件，需要根据集群的实际配置来定制化，我们这里只配置一个单节点的工作集群，配置如下:    

```
node1 ansible_ssh_host=192.168.122.179 ansible_user=root........
[kube-master]
node1

[etcd]
node1

[kube-node]
node1

[k8s-cluster:children]
kube-master
kube-node
```
现在运行以下命令自动化部署集群:    

```
# ansible-playbook -i inventory/test/host.ini cluster.yml
```

部署完毕后的截图如下:    

![/images/2018_10_17_10_18_59_730x528.jpg](/images/2018_10_17_10_18_59_730x528.jpg)

检查kubernetes集群配置情况(工作节点上):    

![/images/2018_10_17_10_19_36_834x334.jpg](/images/2018_10_17_10_19_36_834x334.jpg)

