+++
title = "WorkingTipsOnPrivateHelmRepo"
date = "2018-01-15T09:38:34+08:00"
description = "WorkingTipsOnPrivateHelmRepo"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Steps
Create first chart named `nginxfirst` like following:    

```
# mkdir nginxfirst
# cd nginxfirst/
# ls
# helm create nginxfirst
Creating nginxfirst
# tree
.
└── nginxfirst
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── _helpers.tpl
    │   ├── ingress.yaml
    │   ├── NOTES.txt
    │   └── service.yaml
    └── values.yaml

3 directories, 7 files
```
Edit the values.yaml file:    

```
replicaCount: 1
image:
  repository: mirror.teligen.com/nginx
  tag: 1.7.9
  pullPolicy: IfNotPresent
service:
  name: nginx
  type: ClusterIP
  externalPort: 80
  internalPort: 80
```
Keep others the same.    

`--dry-run` means you want to verificate the configuration.   

Install this chart book via:    

```
# helm install --name firstnginx . --set service.type=NodePort
```
Get the URL via following command:    

```
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services firstnginx-nginxfirst)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```
Finally you will see a running nginx instance.    

### Package and Serve
Package the modified package via following command:    

```
[root@DashSSD nginxfirst]# helm package .
Successfully packaged chart and saved it to: /home/dash/Code/tmp/nginxfirst/nginxfirst/nginxfirst-0.1.0.tgz
[root@DashSSD nginxfirst]# ls
charts  Chart.yaml  nginxfirst-0.1.0.tgz  templates  values.yaml  values.yaml~
[root@DashSSD nginxfirst]# helm lint
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, no failures
```

