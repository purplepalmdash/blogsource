+++
title = "WorkingTipsOnIstioDev"
date = "2018-04-29T11:38:46+08:00"
description = "WorkingTipsOnIstioDev"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Sample SVC
Create a sample svc using minikube:    

```
# sudo docker save jrelva/nginx-autoindex>autoindex.tar
# eval $(minikube docker-env)
# docker load<autoindex.tar
# kubectl run --image=jrelva/nginx-autoindex:latest nginx-autoindex --port=80 --image-pull-policy=IfNotPresent
deployment "nginx-autoindex" created
# kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-autoindex    1         1         1            1           6s
# kubectl expose deployment nginx-autoindex --name nginx-autoindex-svc
# kubectl get svc | grep nginx
nginx-autoindex-svc   ClusterIP   10.107.181.75    <none>        80/TCP           29s
```

Istio Configuration:    

```
# kubectl get svc --all-namespaces | grep istio-ingress
istio-system   istio-ingress          LoadBalancer   10.100.152.241   <pending>     80:30336/TCP,443:32004/TCP  
```
### Istio Ingress

