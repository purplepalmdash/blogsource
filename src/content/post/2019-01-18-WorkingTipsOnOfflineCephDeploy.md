+++
title = "WorkingTipsOnOfflineCephDeploy"
date = "2019-01-18T08:51:48+08:00"
description = "WorkingTipsOnOfflineCephDeploy"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Prerequisites
Offline Deb packages, vagrant boxes(based on ubuntu16.04).     
Put the deb packages onto the webserver and serves as a deb repository.    

### Steps
Get the pip cache for:    

```
# pip download ceph-deploy
### get the ceph-deploy
# ls -l -h
total 676K
-rw-r--r-- 1 root root 113K Jan 18 01:31 ceph-deploy-2.0.1.tar.gz
-rw-r--r-- 1 root root 560K Jan 18 01:31 setuptools-40.6.3-py2.py3-none-any.whl
### Transfer to the offline node
# pip install --no-index --find-links ./ ceph-deploy
# which ceph-deploy
/usr/local/bin/ceph-deploy
```
Edit the host, make the deploy:    

```
# vim /etc/hosts
.....
10.38.129.101	cephdeploy-1
# mkdir ceph-install
# cd ceph-install
# ceph-deploy new cephdeploy-1
```
Edit the python files for supporting offline deployment:    

```
# vim /usr/local/lib/python2.7/dist-packages/ceph_deploy/hosts/remotes.py
def write_sources_list(url, codename, filename='ceph.list', mode=0o644): 
....
....
    #write_file(repo_path, content.encode('utf-8'), mode)
# rm -f /usr/local/lib/python2.7/dist-packages/ceph_deploy/hosts/remotes.pyc
```
Install ceph package:    

```
# ceph-deploy install --repo-url=http://192.xxx.xxx.xxx/cephdeploy --gpg-url \
http://192.xxx.xxx.xxx/cephdeploy/release.asc --release luminous cephdeploy-1
```
Initialize mon:    

```
# ceph-deploy mon create-initial
# ceph-deploy admin cephdeploy-1
```
Now ceph is Ok for accessing via `ceph -s`.    

Deploy ceph mgr:    

```
# ceph-deploy mgr create cephdeploy-1
```
Using ceph-volume lvm for managing disk, so we create the lv(logical volume),
we create 3 lvs for single osd:    

```
# pvcreate /dev/vdb
# vgcreate ceph-pool /dev/vdb
# lvcreate -n osd0.wal -L 1G ceph-pool
# lvcreate -n osd0.db -L 1G ceph-pool
# lvcreate -n osd0 -l 100%FREE ceph-pool
```
Create osd via:    

```
# ceph-deploy  osd create \
    --data ceph-pool/osd0 \
    --block-db ceph-pool/osd0.db \
    --block-wal ceph-pool/osd0.wal \
    --bluestore  cephdeploy-1
```
Now the minimal cluster is ready for use.    

### rbd
Create the osd:    

```
# ceph osd pool create test_pool 128 128 replicated
# rbd crate --size 10240 test_image -p test_pool
# rbd info test_pool/test_image
# ceph osd crush tunables legacy
# rbd feature disable test_pool/test_image exclusive-lock object-map fast-diff \
deep-flatten
# rbd map test_pool/test_image
# mkfs.ext4 /dev/rbd/test_pool/test_image
# mkdir /mnt/ceph-block-device
# chmod 777 /mnt/ceph-block-device/
# mount /dev/rbd/test_pool/test_image /mnt/ceph-block-device
```
