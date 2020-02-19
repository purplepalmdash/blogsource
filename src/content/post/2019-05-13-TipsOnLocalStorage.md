+++
title = "TipsOnLocalStorage"
date = "2019-05-13T17:21:49+08:00"
description = "TipsOnLocalStorage"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Background
For enabling local storage provision on kubespray, and make use of the local
disk for pod storage usage.    

### Enable
Enable the local storage pool via:    

```
# vim inventory/sample/group_vars/k8s-cluster/addons.yml
	# Rancher Local Path Provisioner
	local_path_provisioner_enabled: true
	
	# Local volume provisioner deployment
	local_volume_provisioner_enabled: true
	local_volume_provisioner_namespace: kube-system
	local_volume_provisioner_storage_classes:
	  local-storage:
	    host_dir: /mnt/disks
	    mount_dir: /mnt/disks
	  fast-disks:
	    host_dir: /mnt/fast-disks
	    mount_dir: /mnt/fast-disks
	    block_cleaner_command:
	      - "/scripts/shred.sh"
	      - "2"
	    volume_mode: Filesystem
	    fs_type: ext4
```
### Prepare
Prepare the local storage via:    

```
# 
# mkdir -p  /mnt/fast-disks/vol-alertmanager-res-alertmanager-0
# mkdir -p  /mnt/fast-disks/vol-prometheus-res-prometheus-0
# mkdir -p  /mnt/fast-disks/es-data-es-data-efk-cluster-default-0
# mkdir -p  /mnt/fast-disks/es-data-es-master-efk-cluster-default-0
# truncate /mnt/vol-alertmanager-res-alertmanager-0 --size 20G
# truncate /mnt/vol-prometheus-res-prometheus-0 --size 20G
# truncate /mnt/es-data-es-data-efk-cluster-default-0 --size 10G
# truncate /mnt/es-data-es-master-efk-cluster-default-0 --size 10G
# mkfs.ext4 /mnt/vol-alertmanager-res-alertmanager-0
# mkfs.ext4 /mnt/vol-prometheus-res-prometheus-0
# mkfs.ext4 /mnt/es-data-es-data-efk-cluster-default-0
# mkfs.ext4 /mnt/es-data-es-master-efk-cluster-default-0
```
Edit the `/etc/fstab` for mounting them automatically:    

```
/mnt/vol-alertmanager-res-alertmanager-0	/mnt/fast-disks/vol-alertmanager-res-alertmanager-0 ext4	rw 0	1	
/mnt/vol-prometheus-res-prometheus-0	/mnt/fast-disks/vol-prometheus-res-prometheus-0	ext4	rw	0	1
/mnt/es-data-es-data-efk-cluster-default-0	/mnt/fast-disks/es-data-es-data-efk-cluster-default-0	ext4	rw	0	1
/mnt/es-data-es-master-efk-cluster-default-0	/mnt/fast-disks/es-data-es-master-efk-cluster-default-0	ext4	rw	0	1
```

### Usage
I prepare the storage for I use them in helm/charts, and helm/charts
automatically request the storage from storage class, thus I have to make
`/mnt/fast-disks` as the  default storage class.    

```
# kubectl edit sc fast-disks
kind: StorageClass
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"fast-disks"},"provisioner":"kubernetes.io/no-provisioner","volumeBindingMode":"WaitForFirstConsumer"}
+    storageclass.kubernetes.io/is-default-class: "true"
```

verify:    

```
root@localnode-1:/mnt# kubectl get pvc --all-namespaces
NAMESPACE    NAME                                      STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   AGE
logging      es-data-es-data-efk-cluster-default-0     Bound    local-pv-7d48bf57   20Gi       RWO            fast-disks     4h17m
logging      es-data-es-master-efk-cluster-default-0   Bound    local-pv-64a35d15   20Gi       RWO            fast-disks     4h17m
monitoring   vol-alertmanager-res-alertmanager-0       Bound    local-pv-24ed6560   20Gi       RWO            fast-disks     4h21m
monitoring   vol-prometheus-res-prometheus-0           Bound    local-pv-e998c4c2   20Gi       RWO            fast-disks     4h21m
```
### TBD
1. Now to enlarge the disk?    

