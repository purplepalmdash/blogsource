+++
title = "KismaticGlusterfsTips"
date = "2018-03-08T14:26:40+08:00"
description = "kismatic glusterfs tips"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Configuration
The configuration files is listed as following:    

```
# Worker nodes are the ones that will run your workloads on the cluster.
worker:
  expected_count: 1
  nodes:
  - host: "allinone"
    ip: "10.15.205.93"
    internalip: ""
    labels: {}

storage:
  expected_count: 3
  nodes: 
  - host: "gluster1"
    ip: "10.15.205.90"
    internalip: ""
    labels: {}
  - host: "gluster2"
    ip: "10.15.205.91"
    internalip: ""
    labels: {}
  - host: "gluster3"
    ip: "10.15.205.92"
    internalip: ""
    labels: {}

# A set of NFS volumes for use by on-cluster persistent workloads
nfs:
  nfs_volume: []
```
But it won't startup, the reason is because Ubuntu have a bug of rpcbind,
solved by:    

```
# systemctl add-wants multi-user.target rpcbind.service
# systemctl enable rpcbind.service
# ufw disable
```
Then you should reboot all of the nodes.    

### verification
Create a new glusterfs volume and expose it in k8s as a PV use:    

```
# kismatic volume add 10 storage01 -r 2 -d 1 -c="durable" -a *.*.*.*
```
New PVC:    

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-app-frontend-claim
  annotations:
    volume.beta.kubernetes.io/storage-class: "durable"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

Use pvc in a pod volume:    

```
kind: Pod
apiVersion: v1
metadata:
  name: my-app-frontend
spec:
  containers:
    - name: my-app-frontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: html
  volumes:
    - name: html
      persistentVolumeClaim:
        claimName: my-app-frontend-claim
```
Whenyou scale the pod out, each instance of the pod should have access to that
directory.    
