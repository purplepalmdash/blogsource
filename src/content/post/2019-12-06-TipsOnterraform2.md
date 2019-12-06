+++
title = "用Terraform管理集群编译环境-2"
date = "2019-12-06T15:40:49+08:00"
description = "TipsOnTerraform2"
keywords = ["Linux"]
categories = ["Linux"]
+++
前面已经用terraform可以批量创建出基础环境，但真正要做到集群部署这个环节还是需要有一定的活需要做的。所以后续我将terraform和自己改编的rong揉在了一起。通过预编译好的qcow2镜像，可以快速启动任意个kubernetes节点的集群。   

### 前置条件
qcow2预编译镜像中需安装`cloud-init`, `qemu-guest-agent`两个包。安装完毕后需手动使能`cloud-init`，后续我们在terraform创建虚拟机实例的时候可以通过cloud-init注入一些信息。    

```
# systemctl enable cloud-init
```
debian 9.0上需要安装mkisofs, 因为mkisofs已被genisoimage代替，因而需执行以下操作:     

```
# apt-get install -y genisoimage
# ln -s /usr/bin/genisoimage /usr/bin/mkisofs
```
terraform需要具备以下插件, 其中terraform-provider-libvirt在debian 9.0上需手动编译:    

```
# ls ~/.terraform.d/plugins/
terraform-provider-ansible  terraform-provider-libvirt  terraform-provider-template_v2.1.2_x4
```

### cloud-init文件
`cloud-init.cfg`文件内容如下:     

```
#cloud-config
# https://cloudinit.readthedocs.io/en/latest/topics/modules.html
hostname: ${HOSTNAME}
users:
  - name: xxxxx
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/xxxxx
    shell: /bin/bash
    ssh-authorized-keys:
      - ssh-rsa xxxxxxxxxxxxxxxxxxxx
ssh_pwauth: True
disable_root: false
chpasswd:
  list: |
     xxxxx:linux
  expire: False
```
真正用到的只有`hostname: ${HOSTNAME}`这个变量，其他的步骤是用于创建一个名为`xxxxx`的用户并更改其密码。后续需要对操作系统进行深度定制的时候可以使用该操作。    

### main.tf定义
`main.tf`是整个底层架构编排的核心文件，内容如下:    

```bash 
bash {linenos=table,linenostart=1}
################################################################################
#  vars definition
################################################################################
variable "VM_COUNT" {
  default = 10
  type = number
}

variable "VM_USER" {
  default = "developer"
  type = string
}

variable "VM_HOSTNAME" {
  default = "newnode"
  type = string
}

variable "VM_IMG_URL" {
  default = "http://1xx.xxx.xxx.xxx/xxxx180403cloudinit.img"
  type = string
}

variable "VM_IMG_FORMAT" {
  default = "qcow2"
  type = string
}

# https://www.ipaddressguide.com/cidr
variable "VM_CIDR_RANGE" {
  default = "10.10.10.0/24"
  type = string
}

variable "LIBVIRT_POOL_DIR" {
  default = "./.local/.docker-libvirt"
  type = string
}

#variable libvirt_host {
#  type = string
#  description = "IP address of host running libvirt"
#}
#
#variable instance_name {
#  type = string
#  description = "name of VM instance"
#}

variable pool_name {
  type = string
  default = "default"
  description = "name of pool to store disk and iso image"
}

#variable source_path {
#  type = string
#  description = "path to qcow2 base image, can be remote url or local disk path"
#}

variable disk_format {
  type = string
  default = "qcow2"
}

variable default_password {
  type = string
  default = "passw0rd"
  description = "default password to login to VM when running, it's recommended to disable this manually"
}

variable memory_size {
  type = string
  default = "5120"
  description = "memory size of VM"
}

variable num_cpu {
  default = 2
  description = "number of vCPU which VM has"
}

variable num_network_interface {
  default = 1
  description = "number of network interfaces which VM has"
}

variable private_network_bridge {
  type = string
  default = "virbr0"
  description = "existing network bridge on host that VM needs to connect to private network"
}

variable public_network_bridge {
  type = string
  default = "virbr1"
  description = "existing network bridge on host that VM needs to connect to public network"
}

#variable user_data {
#  type = string
#}

variable autostart {
  default = "true"
  type = string
}

################################################################################
# PROVIDERS
################################################################################

# instance the provider
provider "libvirt" {
  uri = "qemu:///system"
}

# If you want to call remote libvirt provider. 
#provider "libvirt" {
#  uri = "qemu+tcp://${var.libvirt_host}/system"
#}

################################################################################
# DATA TEMPLATES
################################################################################

# https://www.terraform.io/docs/providers/template/d/file.html

# https://www.terraform.io/docs/providers/template/d/cloudinit_config.html
data "template_file" "user_data" {
  count = var.VM_COUNT
  template = file("${path.module}/cloud_init.cfg")
  vars = {
    HOSTNAME = "${var.VM_HOSTNAME}-${count.index + 1}"
  }
}

#data "template_file" "network_config" {
#  template = file("${path.module}/network_config.cfg")
#}


################################################################################
# ANSIBLE ITEMS
################################################################################
resource "ansible_group" "kube-deploy" {
  inventory_group_name = "kube-deploy"
}

resource "ansible_group" "kube-master" {
  inventory_group_name = "kube-master"
}

resource "ansible_group" "kube-node" {
  inventory_group_name = "kube-node"
}

resource "ansible_group" "etcd" {
  inventory_group_name = "etcd"
}

resource "ansible_group" "k8s-cluster" {
  inventory_group_name = "k8s-cluster"
  children = ["kube-master", "kube-node"]
}

# if count > 3, then we have 3 ectds, 3 kube-master, count kube-nodes

# The first node should be kube-deploy/kube-master/kube-node/etcd. 
resource "ansible_host" "deploynode" {
    groups = ["kube-master", "etcd", "kube-node", "kube-deploy"]
    inventory_hostname = "${var.VM_HOSTNAME}-1"
    vars = {
        ansible_user = "root"
        ansible_ssh_private_key_file = "./deploy.key"
        ansible_host = element(libvirt_domain.vm.*.network_interface.0.addresses.0, 0)
        ip = element(libvirt_domain.vm.*.network_interface.0.addresses.0, 0)
    }
    #provisioner "local-exec" {
    #  command = "sleep 40 && ansible-playbook -i  /etc/ansible/terraform.py cluster.yml --extra-vars @rong-vars.yml"
    #}
}

# Create 2(kube-master, etcd, kube-node) nodes, node2, node3
resource "ansible_host" "master" {
    count = var.VM_COUNT >= 3 ? 2 : var.VM_COUNT -1
    groups = var.VM_COUNT >= 3 ? ["kube-master", "etcd", "kube-node"] : ["kube-master", "kube-node"]
    #inventory_hostname = format("%s-%d", "node", count.index + 2)
    inventory_hostname = format("%s-%d", var.VM_HOSTNAME, count.index + 2)
    vars = {
        ansible_user = "root"
        ansible_ssh_private_key_file = "./deploy.key"
        ansible_host = element(libvirt_domain.vm.*.network_interface.0.addresses.0, count.index+1)
        ip = element(libvirt_domain.vm.*.network_interface.0.addresses.0, count.index+1)
    }
}

# others should be kube-nodes
resource "ansible_host" "worker" {
    count = var.VM_COUNT > 3 ? var.VM_COUNT - 3 : 0
    groups = ["kube-node"]
    #inventory_hostname = "node${count.index + 4}"
    #inventory_hostname = format("%s-%d", "node", count.index + 4)
    inventory_hostname = format("%s-%d", var.VM_HOSTNAME, count.index + 4)
    vars = {
        ansible_user = "root"
        ansible_ssh_private_key_file = "./deploy.key"
        ansible_host = element(libvirt_domain.vm.*.network_interface.0.addresses.0, count.index+3)
        ip = element(libvirt_domain.vm.*.network_interface.0.addresses.0, count.index+3)
    }
}

################################################################################
# RESOURCES
################################################################################
resource "libvirt_pool" "vm" {
  name = "${var.VM_HOSTNAME}_pool"
  type = "dir"
  path = abspath("${var.LIBVIRT_POOL_DIR}")
}

# We fetch the disk image for the operating system from the given url. For the base image. 
resource "libvirt_volume" "vm_disk_image" {
  name   = "${var.VM_HOSTNAME}_disk_image.${var.VM_IMG_FORMAT}"
  # Or you could specify like `pool = "transfer"`
  pool   = libvirt_pool.vm.name
  source = var.VM_IMG_URL
  format = var.VM_IMG_FORMAT
}

// It will use the disk image fetched at `libirt_volume.vm_disk_image` as the
//  base one to build the worker VM.
resource "libvirt_volume" "vm_worker" {
  count  = var.VM_COUNT
  name   = "worker_${var.VM_HOSTNAME}-${count.index + 1}.${var.VM_IMG_FORMAT}"
  base_volume_id = libvirt_volume.vm_disk_image.id
  pool   = libvirt_volume.vm_disk_image.pool
}

#*# Create a public network for the VMs
#*# https://www.ipaddressguide.com/cidrv
#*resource "libvirt_network" "vm_public_network" {
#*   name = "${var.VM_HOSTNAME}_network"
#*   autostart = true
#*   mode = "nat"
#*   domain = "${var.VM_HOSTNAME}.local"
#*
#*   # TODO: FIX CIDR ADDRESSES RANGE?
#*   # With `wait_for_lease` enabled, we get an error in the end of the VMs
#*   #  creation:
#*   #   - 'Requested operation is not valid: the address family of a host entry IP must match the address family of the dhcp element's parent'
#*   # But the VMs will be running and accessible via ssh.
#*   addresses = ["${var.VM_CIDR_RANGE}"]
#*
#*   dhcp {
#*    enabled = true
#*   }
#*   dns {
#*    enabled = true
#*   }
#*}

# for more info about paramater check this out 
# https://github.com/dmacvicar/terraform-provider-libvirt/blob/master/website/docs/r/cloudinit.html.markdown
# Use CloudInit to add our ssh-key to the instance
# you can add also meta_data field
resource "libvirt_cloudinit_disk" "cloudinit" {
  count = var.VM_COUNT
  name           = "${var.VM_HOSTNAME}-${count.index + 1}_cloudinit.iso"
  #user_data      = data.template_file.user_data.rendered 
  user_data      = data.template_file.user_data[count.index].rendered
  pool           = libvirt_pool.vm.name
}



resource "libvirt_domain" "vm" {
  count  = var.VM_COUNT
  name   = "${var.VM_HOSTNAME}-${count.index + 1}"
  #memory      = "${var.memory_size}"
  memory      = var.memory_size
  #vcpu        = "${var.num_cpu}"
  vcpu        = var.num_cpu
  #autostart   = "${var.autostart}"
  autostart   = var.autostart

  # TODO: FIX qemu-ga?
  # qemu-ga needs to be installed and working inside the VM, and currently is
  #  not working. Maybe it needs some configuration.
  qemu_agent = true
  #cloudinit = "${libvirt_cloudinit_disk.cloudinit.id}"
  cloudinit = element(libvirt_cloudinit_disk.cloudinit.*.id, count.index)


  # attach network interface to default network(192.168.122.0/24)
  # Or we could specify a new networking created in resource and attached to it. 
  network_interface {
    network_name   = "default"
    hostname   = "${var.VM_HOSTNAME}-${count.index + 1}"
    wait_for_lease = true
  }

  #* Attached to our created network.
  #*network_interface {
  #*  #hostname = "${var.VM_HOSTNAME}-${count.index + 1}"
  #*  network_id = "${libvirt_network.vm_public_network.id}"
  #*  #network_name = "${libvirt_network.vm_public_network.name}"

  #*  #addresses = ["${cidrhost(libvirt_network.vm_public_network.addresses, count.index + 1)}"]
  #*  addresses = ["${cidrhost(var.VM_CIDR_RANGE, count.index + 1)}"]

  #*  # TODO: Fix wait for lease?
  #*  # qemu-ga must be running inside the VM. See notes above in `qemu_agent`.
  #*  wait_for_lease = true
  #*}

  graphics {
    type = "vnc"
    listen_type = "address"
    autoport = true
  }

  # IMPORTANT
  # Ubuntu can hang is a isa-serial is not present at boot time.
  # If you find your CPU 100% and never is available this is why.
  #
  # This is a known bug on cloud images, since they expect a console
  # we need to pass it:
  # https://bugs.launchpad.net/cloud-images/+bug/1573095
  console {
    type        = "pty"
    target_port = "0"
    target_type = "serial"
  }

  console {
    type        = "pty"
    target_type = "virtio"
    target_port = "1"
  }

  disk {
    volume_id = element(libvirt_volume.vm_worker.*.id, count.index)
  }

}
################################################################################
# TERRAFORM CONFIG
################################################################################

terraform {
  required_version = ">= 0.12"
}

################################################################################
# TERRAFORM OUTPUT
################################################################################
#
output "ip" {
  value = "${libvirt_domain.vm.*.network_interface.0.addresses.0}"
}
```
逐行解释如下:    
