+++
title = "WorkingTipsOnIstioDemo"
date = "2018-05-16T10:42:55+08:00"
description = "WorkingTipsOnIstioDemo"
keywords = ["Linux"]
categories = ["Linux"]
+++
### istio 0.8 download
Download from its daily build:    

[https://gcsweb.istio.io/gcs/istio-prerelease/daily-build/](https://gcsweb.istio.io/gcs/istio-prerelease/daily-build/)    

```
# wget https://storage.googleapis.com/istio-prerelease/daily-build/release-0.8-20180515-17-26/istio-release-0.8-20180515-17-26-linux.tar.gz
# mkdir -p /root/istio/bin/
# cp /root/Code/istio-release-0.8-20180515-17-26/bin/istioctl /root/istio/bin/
```

### Build
Install following packages:    

```
# yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel java-1.8.0-openjdk-debug
# export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64
# export PATH=$PATH:$JAVA_HOME/bin
```
Build the cars-api service via:    

```
# ./mvnw -Distio.home=/root/Code/istio-release-0.8-20180515-17-26 clean package fabric8:build
# docker images | grep cars-api
kameshsampath/cars-api                                                      0.0.1               28647076e814        8 minutes ago       439 MB
```

### Verification
Install cars-api:    

```
# kubectl  apply -f istio-cars-api-0.0.1-all.yml 
deployment.extensions "cars-api" created
# kubectl  get pods
NAME                                      READY     STATUS    RESTARTS   AGE
cars-api-777b9574bf-jvxvk                 2/2       Running   0          3m
```
`car-api-ingress.yaml` definition:    

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  labels:
    expose: "true"
    app: cars-api
    version: 0.0.1
  name: cars-api
  namespace: default
  annotations:
    kubernetes.io/ingress.class: istio
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: cars-api
          servicePort: 8080
```

Create a ingress:    

```
# kubectl create -f car-api-ingress.yaml
```

Test the car api via:    

```
# curl -vvv http://192.192.189.41:32204/cars/list
```

### auth
Definition file `auth.yaml`:    

```
apiVersion: "authentication.istio.io/v1alpha1"
kind: "Policy"
metadata:
  name: "cars-api"
spec:
  targets:
  - name: cars-api
  peers:
  - mtls:
  origins:
  - jwt:
      issuer: http://keycloak.default:8080/auth/realms/istio
      jwksUri: http://keycloak.default:8080/auth/realms/istio/protocol/openid-connect/certs
      audiences: 
      - cars-web  
  principalBinding: USE_ORIGIN
```

Create the Policy via:    

```
# /root/istio/bin/istioctl create  -f auth.yaml
```
Now re-visit the ingress item you will see 401 issue:    

```
# curl -vvv http://192.192.189.41:32204/cars/list

*   Trying 192.192.189.41...
* TCP_NODELAY set
* Connected to 192.192.189.41 (192.192.189.41) port 32204 (#0)
> GET /cars/list HTTP/1.1
> Host: 192.192.189.41:32204
> User-Agent: curl/7.59.0
> Accept: */*
> 
< HTTP/1.1 401 Unauthorized
< content-length: 29
< content-type: text/plain
< date: Wed, 16 May 2018 03:16:47 GMT
< server: envoy
< x-envoy-upstream-service-time: 8
< 
* Connection #0 to host 192.192.189.41 left intact
Origin authentication failed.%              
```
Get the token, then :    

```
# curl -vvv -H "Authorization: Bearer $token" http://192.192.189.41:32204/cars/list
```
you will  get the right result. 

### Different Namespace
Create the auth.yaml for different namespace is OK:    

```
# /root/istio/bin/istioctl create -f auth-myproject.yaml  -n myproject
```
