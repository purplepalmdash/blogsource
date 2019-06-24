+++
title = "WorkingTipsOnArmKubespray"
date = "2019-06-24T08:36:19+08:00"
description = "WorkingTipsOnArmKubespray"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Emulation
Install uml-utilites for arm emulator networking:    

```
# sudo apt-get install -y uml-utilities
# sudo tunctl -u  dash -t tap0
# sudo ifconfig tap0 192.168.101.1 up
# echo 1 > /proc/sys/net/ipv4/ip_forward
# iptables -t nat -A POSTROUTING -o enp0s20f0u12u1 -j MASQUERADE
# iptables -I FORWARD 1 -i tap0 -j ACCEPT
# iptables -I FORWARD 1 -o tap0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```
Now your tap0 will be ready for working.   

Install qemu for aarch64:     

```
# apt-get install qemu-system-arm
# which qemu-system-aarch64
/usr/bin/qemu-system-aarch64
```

Using emulator via:    

```
qemu-system-aarch64 -m 6024 -cpu cortex-a57  -M virt -nographic -pflash flash0.img -pflash flash1.img -drive if=none,file=bionic-server-cloudimg-arm64.img,id=hd0 -device virtio-blk-device,drive=hd0 -drive file=user-data.img,format=raw -net nic -net tap,ifname=tap0,script=no #-device virtio-net-device,netdev=net0,mac=52:54:00:12:34:56
```

Too slow on x86. So I switches to arm based server.   

### WorkingOnARMServer
Mount iso and use it as the repository server:    

```
#  mount -t iso9660 -o loop ubuntu-18.04.2-server-arm64.iso /mnt
# apt-cdrom -d=/mnt add
#  cat /etc/apt/sources.list
deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release arm64 (20190210)]/ bionic main r
estricted
# apt-get update -y
# cp ubuntu-18.04.2-server-arm64.iso ubuntu-18.04.2-server-arm64-1.iso
# mount -t iso9660 -o loop ./ubuntu-18.04.2-server-arm64-1.iso /media/cdrom
```
Install some packages for qemu:    

```
# apt-get install -y qemu-kvm build-essential 
```
When offline, we use the offline repository:    

```
# cd /root/armdebs
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
# vim /etc/apt/sources.list
deb [trusted=yes] file:///home/test/armdebs ./
# apt-get update -y
# apt-get install -y uml-utilities
```
From now on we could use qemu-kvm for accessing our vm.    

Run emulated vms and install kubernetes, successful.    

### Migration
Notice, you should run following(pip download) on arm based machine!!!    
On phsical machine, do following steps:    

```
# apt-get install -y python-pip
# mkdir piparmarm
# cd piparmarm/
# pip download ansible
# pip download httplib2
```
Transfer the `piparmarm` folder to your machine, install it via:    

```
root@arm01:~/piparmarm# pip install --no-index --find-links /home/test/piparmarm/ ansible httplib2
root@arm01:~/piparmarm# which ansible
/usr/local/bin/ansible
root@arm01:~/piparmarm# ansible --version
ansible 2.8.1
  config file = None
  configured module search path = [u'/home/test/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python2.7/dist-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 2.7.15rc1 (default, Nov 12 2018, 14:31:15) [GCC 7.3.0]
```
Install docker-ce(manually):     

```
# apt-get install -y docker-ce python-netaddr
# docker load<allimages.tar
```
Configure the hosts.ini, then run playbook:    

```
# ansible-playbook -i inventory/sample/hosts.ini cluster.yml
```

