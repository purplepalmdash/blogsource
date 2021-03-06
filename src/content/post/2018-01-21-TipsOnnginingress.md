+++
title = "tipsonnginxingress"
date = "2018-01-21T14:24:12+08:00"
description = "tipsonnginxingress"
keywords = ["Linux"]
categories = ["Technology"]
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

### Config examples
For monocular, do following:    

```
# vim custom-domains.yaml
ingress:
  hosts:
  - monocular.xxxxxx.com
# helm install --name=gou -f custom-domains.yaml .
```
Now adding monocular.xxxxxx.com into your `/etc/hosts` file, you could
directly access monocular UI using domain name.    


For wordpress:    

```
# vim values.yaml
ingress:
  ## Set to true to enable ingress record generation
  enabled: true

  ## The list of hostnames to be covered with this ingress record.
  ## Most likely this will be just one host, but in the event more hosts are needed, this is an array
  hosts:
  - name: wordpress.xxxxxx.com

```
The same as before, you could use wordpress.xxxxxx.com for accessing
wordpress.     
