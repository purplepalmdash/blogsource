+++
title = "WorkingTipsOnCephDeploy"
date = "2019-01-14T21:29:45+08:00"
description = "WorkingTipsOnCephDeploy"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Environment
Ubuntu16.04, IPs are listed as following:    

```
cephdeploy-1	10.28.129.101
cephdeploy-2	10.28.129.102
cephdeploy-3	10.28.129.103
cephdeploy-4	10.28.129.104
```
### Configure apt sources
Doing via ansible play-books:    

```
- hosts: all
  gather_facts: false
  become: True
  tasks:
    - name: "Run shell"
      shell: uptime 

    - name: "Configure apt sources"
      shell: rm -f /etc/apt/sources.list && echo "deb http://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse">/etc/apt/sources.list && echo "deb http://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse">>/etc/apt/sources.list && echo "deb http://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse">>/etc/apt/sources.list && echo "deb http://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse">>/etc/apt/sources.list && echo "deb http://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse">>/etc/apt/sources.list && apt-get update -y

    - name: "Add Ceph User"
      raw: useradd -d /home/cephuser -m cephuser && echo "cephuser ALL = (root) NOPASSWD:ALL" | tee /etc/sudoers.d/cephuser  && chmod 0440 /etc/sudoers.d/cephuser

    - name: "Change password"
      raw: usermod -p '$1$5RPVAd$kC4MwCLFLL2j7MBLgWv.H.' cephuser

    - name: "Add ceph repository"
      raw:  wget -q -O- 'http://mirrors.163.com/ceph/keys/release.asc' | sudo apt-key add - && echo deb http://mirrors.163.com/ceph/debian-luminous/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

    - name: "Install python"
      shell: apt-get install -y python
```
The password is generated via following method:    

```
#  openssl passwd -1 -salt 5RPVAd clear-text-passwd43
$1$5RPVAd$vgsoSANybLDepv2ETcUH7.
```
### Ceph-Deploy
Roles for ceph cluster:    

```
cephdeploy-1	10.28.129.101 ceph-admin
cephdeploy-2	10.28.129.102 ceph-mon
cephdeploy-3	10.28.129.103 osd-server-1
cephdeploy-4	10.28.129.104 osd-server-2
```
Ssh into cephdeploy-1, do following:     

```
# apt-get install -y python-pip
# pip install ceph-deploy
```
Generate ssh key via and configure password-less login:    

```
# ssh-keygen
# vim /etc/hosts
10.28.129.102	cephdeploy-2
10.28.129.103	cephdeploy-3
10.28.129.104	cephdeploy-4
# ssh-copy-id cephuser@cephdeploy-2
# ssh-copy-id cephuser@cephdeploy-3
# ssh-copy-id cephuser@cephdeploy-4
# vim ~/.ssh/config
Host cephdeploy-2
  Hostname cephdeploy-2
  User cephuser
Host cephdeploy-3
  Hostname cephdeploy-3
  User cephuser
Host cephdeploy-4
  Hostname cephdeploy-4
  User cephuser
```
Make ceph-deploy folder and generate configuration files:    

```
# mkdir ~/my-cluster
# cd ~/my-cluster/
# ceph-deploy new cephdeploy-2
```
Modify the configuration file:     

```
# vim ceph.conf
.......
osd pool default size = 2
osd journal size = 2000
public network = 10.28.129.0/24
cluster network = 10.28.129.0/24

```

Install ceph via following command:    

```
# export CEPH_DEPLOY_REPO_URL=http://mirrors.163.com/ceph/debian-luminous/
# export CEPH_DEPLOY_GPG_URL=http://mirrors.163.com/ceph/keys/release.asc
# ceph-deploy install cephdeploy-1 cephdeploy-2 cephdeploy-3 cephdeploy-4
```
Create initial mon:    

```
# ceph-deploy mon create-initial
```
Create osd via:      

```
# ceph-deploy disk zap cephdeploy-3 /dev/vdb
# ceph-deploy disk zap cephdeploy-4 /dev/vdb
# ceph-deploy osd create cephdeploy-3 --data /dev/vdb
# ceph-deploy osd create cephdeploy-4 --data /dev/vdb
```
You could examine the osd via:     

```
# ceph-deploy osd list cephdeploy-3 
# ceph-deploy osd list cephdeploy-4
```
Create admin for :     

```
# ceph-deploy admin cephdeploy-1 cephdeploy-2 cephdeploy-3 cephdeploy-4
# sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

### Configure ceph
Examine the ceph health via:    

```
ceph -s
  cluster:
    id:     1674bddc-65c1-40c5-8f88-f18aef7a3d32
    health: HEALTH_WARN
            no active mgr
```
That's because we lost active mgr, create one via:     

```
# ceph-deploy mgr create cephdeploy-2:mon_mgr
# ceph -s
  cluster:
    id:     1674bddc-65c1-40c5-8f88-f18aef7a3d32
    health: HEALTH_OK
# ceph health
HEALTH_OK
```
Enable the dashboard via:     

```
# ceph mgr module enable dashboard
```
So you could open your browser to `http://10.28.129.102:7000`, you could reach
ceph dashboard.    

![/images/2019_01_14_22_30_12_878x605.jpg](/images/2019_01_14_22_30_12_878x605.jpg)

### rbd
Create and configure:     

```
# ceph osd pool create test_pool 128 128 replicated
# ceph osd lspools
# rbd create --size 10240 test_image -p test_pool
# rbd info test_pool/test_image
# rbd feature disable test_pool/test_image exclusive-lock object-map fast-diff deep-flatten
# apt-get install rbd-nbd
# rbd-nbd map test_pool/test_image
/dev/nbd0

```

Or if you would not use rbd-nbd, then you could use following commands:    

```
# ceph osd crush tunables legacy
# rbd map test_pool/test_image
```
