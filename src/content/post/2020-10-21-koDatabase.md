+++
title= "koDatabase"
date = "2020-10-21T11:15:23+08:00"
description = "koDatabase"
keywords = ["Technology"]
categories = ["Technology"]
+++
### ko admin
Via following commands for recoving the user priviledge:    

```
# podman exec -it rong_mysql /bin/bash
```
Sql 操作:    

```
mysqlbash-4.2# mysql -uroot -p
Enter password: 
mysql> use ko
mysql> update ko_user set is_active=1 where name='admin';
mysql> update ko_user set is_admin=1 where name='admin;
```
Now  go back to login page, you will use admin user for login.

### ko cluster import
Import cluster to ko:   

```
root@focal-1:/mnt/Rong_RongGraph/rong/4_addons# kubectl get sa -n kube-system | grep dashboard
kubernetes-dashboard                 1         10m
root@focal-1:/mnt/Rong_RongGraph/rong/4_addons# kubectl get secret -n kube-system | grep dashboard
kubernetes-dashboard-certs                       Opaque                                0      10m
kubernetes-dashboard-csrf                        Opaque                                1      10m
kubernetes-dashboard-key-holder                  Opaque                                2      10m
kubernetes-dashboard-token-mpf77                 kubernetes.io/service-account-token   3      10m
root@focal-1:/mnt/Rong_RongGraph/rong/4_addons# kubectl -n kube-system describe secrets kubernetes-dashboard-token-mpf77
Name:         kubernetes-dashboard-token-mpf77
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: ff6cac3e-d90c-4990-bb90-e245ac762696

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      xxxxxxx

```
