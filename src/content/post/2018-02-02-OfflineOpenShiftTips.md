+++
title = "OfflineOpenshiftTips"
date = "2018-02-02T20:53:53+08:00"
description = "OfflineOpenshiftTips"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Environment
Create a new isolated network in virt-manager:    

![/images/2018_02_02_20_54_54_411x189.jpg](/images/2018_02_02_20_54_54_411x189.jpg)

Specify the cidr of this network:    

![/images/2018_02_02_20_55_54_413x272.jpg](/images/2018_02_02_20_55_54_413x272.jpg)

specify isolated:    

![/images/2018_02_02_20_58_50_400x231.jpg](/images/2018_02_02_20_58_50_400x231.jpg)

### Virtual Machine
Two nodes, created via qemu-img:    

```
# qemu-img create -f qcow2 -b base/centos74.qcow2 OpenShiftMaster.qcow2
# qemu-img create -f qcow2 -b base/centos74.qcow2 OpenShiftNode1.qcow2
```
Master node: 

![/images/2018_02_02_21_06_26_368x212.jpg](/images/2018_02_02_21_06_26_368x212.jpg)

![/images/2018_02_02_21_09_05_435x206.jpg](/images/2018_02_02_21_09_05_435x206.jpg)

![/images/2018_02_02_21_09_35_411x182.jpg](/images/2018_02_02_21_09_35_411x182.jpg)


Node node1: 

![/images/2018_02_02_21_07_30_368x216.jpg](/images/2018_02_02_21_07_30_368x216.jpg)

![/images/2018_02_02_21_10_35_476x188.jpg](/images/2018_02_02_21_10_35_476x188.jpg)

![/images/2018_02_02_21_10_56_405x164.jpg](/images/2018_02_02_21_10_56_405x164.jpg)

### Repository Preparation
Mount the iso for becomming the iso repo:   

```
# mkdir isopkgs
# mount -t iso9660 -o loop ./CentOS-7-x86_64-Everything-1708.iso isopkgs 
mount: /media/sda/offlineopenshift/isopkgs: WARNING: device write-protected, mounted read-only.
# mkdir centos74iso
# cp -ar isopkgs/* centos74iso
```

![/images/2018_02_02_21_22_23_449x272.jpg](/images/2018_02_02_21_22_23_449x272.jpg)

![/images/2018_02_02_21_22_42_479x232.jpg](/images/2018_02_02_21_22_42_479x232.jpg)

Thus your repository will be:    

```
# vim all.repo 
[iso]
name=iso
baseurl=http://192.168.0.100/offlineopenshift/centos74iso/
enabled=1
gpgcheck=0

[local]
name=repo
baseurl=http://192.168.0.100/offlineopenshift/mypkgs/
enabled=1
gpgcheck=0
```
To upload this file to both master and node's `/etc/yum.repos.d/`, and remove
all of the other repos:   

```
[root@node1 yum.repos.d]# ls
all.repo  back
# yum makecache
```

### Networking For Nodes
Add both for master and node1, and enable password-less login for master/node1:    

```
# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.33.34.2      master
10.33.34.3      node1
# ssh-copy-id root@master
# ssh-copy-id root@node1
```

### master
Install ansible for deploying the offline openshift:    

```
# yum install -y ansible
```
Edit the hosts configuration:    

```
# vim /etc/ansible/hosts
	############################# openshift ##############################3
	# Create an OSEv3 group that contains the masters, nodes, and etcd groups
	[OSEv3:children]
	masters
	nodes
	etcd
	 
	# Set variables common for all OSEv3 hosts
	[OSEv3:vars]
	ansible_ssh_user=root
	deployment_type=origin
	openshift_disable_check=docker_image_availability,docker_storage,memory_availability
	
	[masters]
	master
	 
	# host group for etcd
	[etcd]
	node1
	 
	# host group for nodes, includes region info
	[nodes]
	master openshift_node_labels="{'region': 'infra', 'zone': 'default'}" openshift_schedulable=true
	node1 openshift_node_labels="{'region': 'infra', 'zone': 'default'}" openshift_schedulable=true
```
Test the connectivity:    

```
# ansible all -m ping
master | SUCCESS => {
    "changed": false, 
    "failed": false, 
    "ping": "pong"
}
node1 | SUCCESS => {
    "changed": false, 
    "failed": false, 
    "ping": "pong"
}
```
Checkout the , and edit the yml for disabling the repository updating:    

```
# vim ./roles/openshift_repos/tasks/centos_repos.yml
- name: Configure origin gpg keys
  copy:
    src: "origin/gpg_keys/openshift-ansible-CentOS-SIG-PaaS"
    dest: "/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-PaaS"
  notify: refresh cache

# openshift_release is formatted to a standard string in openshift_version role.
# openshift_release is expected to be in format 'x.y.z...' here.
# Here, we drop the '.' characters and try to match the correct repo template
# for our corresponding openshift_release.
#- name: Configure correct origin release repository
#  template:
#    src: "{{ item }}"
#    dest: "/etc/yum.repos.d/{{ (item | basename | splitext)[0] }}"
#  with_first_found:
#    - "CentOS-OpenShift-Origin{{ (openshift_release | default('')).split('.') | join('') }}.repo.j2"
#    - "CentOS-OpenShift-Origin{{ ((openshift_release | default('')).split('.') | join(''))[0:2] }}.repo.j2"
#    - "CentOS-OpenShift-Origin.repo.j2"
#  notify: refresh cache
```
Ansible play magic:    

```
# cd openshift-ansible
# ansible-playbook playbooks/byo/config.yml
```
