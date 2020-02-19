+++
title = "NFSStorageForK8s"
date = "2018-05-14T21:31:41+08:00"
description = "NFSStorageForK8s"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Code
Code is git from following url:    

```
# git clone https://github.com/kubernetes-incubator/external-storage.git
```
### Steps
You have to manually get the docker image for
`quay.io/external_storage/nfs-client-provisioner`, then create the storage
class via following command:    

deployment.yaml:    

```
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 192.192.189.31
            - name: NFS_PATH
              value: /opt/nfs
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.192.189.31
            path: /opt/nfs
```
Install via `kubectl create -f deployment.yaml`.    

StorageClass yaml is listed as:    

sc.yaml:    

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: fuseim.pri/ifs
```

Create it via `kubectl create -f sc.yaml`.    

test-claim.yaml:    

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
  annotations:
    volume.beta.kubernetes.io/storage-class: "nfs-storage"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

And test-pod.yaml:   

```
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: gcr.io/google_containers/busybox:1.24
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-claim
```

Verify them via install and delete them.   
