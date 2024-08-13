+++
title= "WorkingTipsOnIngressSpice"
date = "2024-08-13T17:39:53+08:00"
description = "WorkingTipsOnIngressSpice"
keywords = ["Technology"]
categories = ["Technology"]
+++
Load and push images:     

```
 nerdctl load<nginxslim.tar
 nerdctl tag gcr.io/google_containers/nginx-slim:0.8 localhost:35000/nginx-slim:0.8
 nerdctl push localhost:35000/nginx-slim:0.8
```
Create the deployment:    

```
# cat nginx01.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx01
  name: nginx01
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx01
  template:
    metadata:
      labels:
        app: nginx01
    spec:
      containers:
      - image: 192.168.1.11:35000/nginx-slim:0.8
        name: nginx01
# kubectl create -f nginx01.yaml
# kubectl expose deployment nginx01 --name=nginx01-svr --type=ClusterIP --port=80
```
Create ingress:     

```
# cat ingress_nginx.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: securebrowser.example
      http:
        paths:
          - path: /nginx
            pathType: Prefix
            backend:
              service:
                name: nginx01-svr
                port:
                  number: 80
# kubectl create -f ingress_nginx.yaml
```
Test:    

```
$ cat /etc/hosts  | grep secure
192.168.1.11	securebrowser.example
$ curl securebrowser.example/nginx
```
