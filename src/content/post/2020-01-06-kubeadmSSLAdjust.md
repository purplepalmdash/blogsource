+++
title = "KubeadmSSLAdjust"
date = "2020-01-06T17:24:47+08:00"
description = "kubeadmssladjust"
keywords = ["Linux"]
categories = ["Linux"]
+++

### 检查
一年过期的集群（即将过期):    

```
root@node:/home/test# cd /etc/kubernetes/ssl
root@node:/etc/kubernetes/ssl# for i in `ls *.crt`; do openssl x509 -in $i -noout -dates; done | grep notAfter
notAfter=Jun 13 02:57:27 2020 GMT
notAfter=Jun 13 02:57:28 2020 GMT
notAfter=Jun 11 02:57:27 2029 GMT
notAfter=Jun 11 02:57:28 2029 GMT
notAfter=Jun 13 02:57:29 2020 GMT
```
获得该集群信息:    

```
# kubectl get nodes -o wide
NAME           STATUS   ROLES    AGE    VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-master-1   Ready    master   206d   v1.12.4   192.192.185.63   <none>        Ubuntu 16.04.4 LTS   4.4.0-116-generic   docker://18.6.1
k8s-master-2   Ready    node     89d    v1.12.4   192.192.185.64   <none>        Ubuntu 16.04.4 LTS   4.4.0-116-generic   docker://18.6.1
k8s-node-1     Ready    node     111d   v1.12.4   192.192.185.65   <none>        Ubuntu 16.04.6 LTS   4.15.0-45-generic   docker://18.6.1
k8s-node-2     Ready    node     206d   v1.12.4   192.192.185.66   <none>        Ubuntu 16.04.4 LTS   4.4.0-116-generic   docker://18.6.1
k8s-node-3     Ready    node     201d   v1.12.4   192.192.189.61   <none>        Ubuntu 16.04.4 LTS   4.4.0-116-generic   docker://18.6.1
```
renew apiserver certs:    

```
# kubeadm alpha phase certs renew apiserver --config=/etc/kubernetes/kubeadm-config.v1alpha3.yaml
#  for i in `ls *.crt`; do openssl x509 -in $i -noout -dates; done | grep notAfter
notAfter=Jun 13 02:57:27 2020 GMT
notAfter=Jan  5 09:30:17 2021 GMT
notAfter=Jun 11 02:57:27 2029 GMT
notAfter=Jun 11 02:57:28 2029 GMT
notAfter=Jun 13 02:57:29 2020 GMT

```


```
# cd /etc/kubernetes/

# ln -s ssl pki

# kubeadm alpha phase certs renew apiserver --config=/etc/kubernetes/kubeadm-config.v1alpha3.yaml --cert-dir=/etc/kubernetes/ssl

# kubeadm alpha phase certs renew apiserver-kubelet-client  --config=/etc/kubernetes/kubeadm-config.v1alpha3.yaml --cert-dir=/etc/kubernetes/ssl

# kubeadm alpha phase certs renew front-proxy-client --config=/etc/kubernetes/kubeadm-config.v1alpha3.yaml --cert-dir=/etc/kubernetes/ssl
```

```
root@node:/etc/kubernetes/ssl# !2036
for i in `ls *.crt`; do openssl x509 -in $i -noout -dates; done | grep notAfter
notAfter=Jan  5 09:36:10 2021 GMT
notAfter=Jan  5 09:30:46 2021 GMT
notAfter=Jun 11 02:57:27 2029 GMT
notAfter=Jun 11 02:57:28 2029 GMT
notAfter=Jan  5 09:37:14 2021 GMT

```


```
kubeadm alpha phase kubeconfig all --apiserver-advertise-address=192.192.185.63
```
