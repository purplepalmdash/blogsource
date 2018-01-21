+++
title = "tipsonnginxingress"
date = "2018-01-21T14:24:12+08:00"
description = "tipsonnginxingress"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Create service
Create a service of nginx via following steps:    

```
# kubectl run my-nginx --image=nginx --replicas=2 --port=80
# kubectl expose deployment my-nginx --port=8080 --target-port=80 --external-ip=x.x.x.168  
```
Verify the service via:    

```
kubectl get svc | grep my-nginx
my-nginx                             ClusterIP      172.20.236.170   10.15.205.200   8080/TCP                     3h
```
### nginx-ingress
Use helm for installing nginx-ingress

```
# helm fetch stable/nginx-ingress
# tar xzvf nginx-ingress-0.8.23.tgz
# cd nginx-ingress
```
Made modifications to following items:    

```
controller:

//........

	hostNetwork: true
```
Because in our environment we have to use hostNetwork for a real ip address
mapping.   

### expose to nginx
Define an yaml via:    

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-kibana-ingress
  namespace: default
spec:
  rules:
  - host: ubuntu.xxxxx.com
    http:
      paths:
      - backend:
          serviceName: my-nginx
          servicePort: 8080
```
Visit your `ubuntu.xxxxx.com`, you will get the nginx service webpage.   
