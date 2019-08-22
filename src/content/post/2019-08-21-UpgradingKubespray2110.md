+++
title = "UpgradingKubespray2110"
date = "2019-08-21T11:08:22+08:00"
description = "UpgradingKubespray2110"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Prepare
vagrant vms, install kubespray via vagrant.     

Additional packages:    

```
# apt-add-repository ppa:ansible/ansible
# sudo apt-get install -y netdata ntpdate ansible python-netaddr bind9 bind9utils ntp nfs-common nfs-kernel-server python-netaddr nethogs iotop
# sudo mkdir /root/static
# cd /var/cache
# find . | grep deb$ | xargs -I % cp % /root/stati
# cd /root/static
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
# cd /root/
# tar cJvf 1804debs.tar.xz static/
```
scp the 1804debs.tar.xz to the `kube-deploy` role(later).

### Modification


```
# wget https://github.com/projectcalico/calicoctl/releases/download/v3.7.3/calicoctl-linux-amd64
# wget https://github.com/containernetworking/plugins/releases/download/v0.8.1/cni-plugins-linux-amd64-v0.8.1.tgz
```
