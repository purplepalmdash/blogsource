+++
title = "用Terraform管理集群编译环境"
date = "2019-12-03T10:30:21+08:00"
description = "UsingTerraform"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 环境
操作系统Ubuntu18.04.3    
libvirtd (libvirt) 4.0.0

### 迅速搭建
terraform下载并加入到系统目录:    

```
$ wget https://releases.hashicorp.com/terraform/0.12.17/terraform_0.12.17_linux_amd64.zip
$  unzip terraform_0.12.17_linux_amd64.zip
$ sudo mv terraform /usr/bin
$ terraform version
Terraform v0.12.17
```
terraform-provider-libvirt下载并完成初始化(`https://github.com/dmacvicar/terraform-provider-libvirt/releases`):    

```
$ wget https://github.com/dmacvicar/terraform-provider-libvirt/releases/download/v0.6.0/terraform-provider-libvirt-0.6.0+git.1569597268.1c8597df.Ubuntu_18.04.amd64.tar.gz
$ tar xzvf terraform-provider-libvirt-0.6.0+git.1569597268.1c8597df.Ubuntu_18.04.amd64.tar.gz
$  terraform init
Terraform initialized in an empty directory!

The directory has no Terraform configuration files. You may begin working
with Terraform immediately by creating Terraform configuration files.
$ cd ~/.terraform.d
$ mkdir plugins
$ cp terraform-provider-libvirt plugins/
```
### 创建第一个环境
创建工作目录:      

```
$ mkdir ~/projects/terraform
$ cd ~/projects/terraform
```
创建一个名为`libvirt.tf`的定义文件，定义在kvm上需要创建的虚拟机:    

```
provider "libvirt" {
  uri = "qemu:///system"
}

resource "libvirt_volume" "node1-qcow2" {
  name = "node1-qcow2"
  pool = "default"
  source = "/media/sda/rong_ubuntu_180403.qcow2"
  format = "qcow2"
}

# Define KVM domain to create
resource "libvirt_domain" "node1" {
  name   = "node1"
  memory = "10240"
  vcpu   = 2

  network_interface {
    network_name = "default"
  }

  disk {
    volume_id = libvirt_volume.node1-qcow2.id
  }

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}
```
初始化一个terraform工作目录, 而后生成并展示terraform执行计划，而后创建定义好的底层设施:      

```
$ terraform init
$ terraform plan
$ terraform apply
```
销毁:     

```
$ terraform destroy 
```
在apply和destroy时需要回答`yes`，如果需要跳过确认流程，则使用以下命令:     

```
$ terraform apply -auto-approve
$ terraform destroy -auto-approve
```
### cloud-init
这个可以参考example下ubuntu的例子。

### multiple vms
参考样例如下:      

```
provider "libvirt" {
  uri = "qemu:///system"
}

variable "hosts" {
  default = 2
}

variable "hostname_format" {
  type    = string
  default = "node%02d"
}

resource "libvirt_volume" "node-disk" {
  name             = "node-${format(var.hostname_format, count.index + 1)}.qcow2"
  count            = var.hosts
  base_volume_name = "xxxxx180403_vagrant_box_image_0.img"
  pool             = "default"
  format           = "qcow2"
}

resource "libvirt_domain" "node" {
  count  = var.hosts
  name   = format(var.hostname_format, count.index + 1)
  vcpu   = 1
  memory = 2048

  disk {
    volume_id = element(libvirt_volume.node-disk.*.id, count.index)
  }

  network_interface {
    network_name   = "default"
    mac            = "52:54:00:00:00:a${count.index + 1}"
    wait_for_lease = true
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}

terraform {
  required_version = ">= 0.12"
}
```
值得注意的是，该定义文件中使用了dhcp地址绑定，为此我们需要定义如下的dhcp规则：     

```
$ sudo virsh net-dumpxml --network default
<network>
  <name>default</name>
  <uuid>c71715ac-90b5-483a-bb1c-6a40a5af1b56</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:92:5c:47'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
      <host mac='52:54:00:00:00:a1' name='node01' ip='192.168.122.171'/>
      <host mac='52:54:00:00:00:a2' name='node02' ip='192.168.122.172'/>
      <host mac='52:54:00:00:00:a3' name='node03' ip='192.168.122.173'/>
      <host mac='52:54:00:00:00:a4' name='node04' ip='192.168.122.174'/>
      <host mac='52:54:00:00:00:a5' name='node05' ip='192.168.122.175'/>
      <host mac='52:54:00:00:00:a6' name='node06' ip='192.168.122.176'/>
      <host mac='52:54:00:00:00:a7' name='node07' ip='192.168.122.177'/>
      <host mac='52:54:00:00:00:a8' name='node08' ip='192.168.122.178'/>
      <host mac='52:54:00:00:00:a9' name='node09' ip='192.168.122.179'/>
      <host mac='52:54:00:00:00:aa' name='node10' ip='192.168.122.180'/>
    </dhcp>
  </ip>
</network>
```
重新定义的规则如下:      

```
$ sudo virsh net-dumpxml --network default>default.xml
修改
$ sudo virsh net-define ./default.xml
重新检查规则
$ sudo virsh net-dumpxml --network default
```
定义完毕以后，则我们在tf文件中定义的虚拟机会通过DHCP从default网络得到相应的IP地址，有利于后续的集群部署。    

检查IP是否被分配的命令:       

```
$ sudo virsh net-dhcp-leases default
 Expiry Time          MAC address        Protocol  IP address                Hostname        Client ID or DUID
-------------------------------------------------------------------------------------------------------------------
 2019-12-03 15:53:49  52:54:00:00:00:a1  ipv4      192.168.122.171/24        node01          01:52:54:00:00:00:a1
 2019-12-03 15:53:49  52:54:00:00:00:a2  ipv4      192.168.122.172/24        node02          01:52:54:00:00:00:a2
```
定义完该网络后，需要手动重启该网络才可以使得更改生效。    
