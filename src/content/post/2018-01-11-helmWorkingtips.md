+++
title = "helmWorkingtips"
date = "2018-01-11T11:51:36+08:00"
description = "helmWorkingtips"
keywords = ["Linux"]
categories = ["Linux"]
+++
### minikube
Install and initialization:    

```
$ sudo cp /media/sda5/kismatic/allinone/helm /usr/bin
$ sudo chmod 777 /usr/bin/helm
$ helm version
Client: &version.Version{SemVer:"v2.7.2", GitCommit:"8478fb4fc723885b155c924d1c8c410b7a9444e6", GitTreeState:"clean"}
Error: cannot connect to Tiller
$ helm init
$HELM_HOME has been configured at /home/xxxx/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
Happy Helming!
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
$ helm ls
$ helm search
```
### monocular
In minikube, we should use hostNetwork mode:    

Prerequisites:    

```
$ helm install stable/nginx-ingress --set controller.hostNetwork=true
```
If on kismatic, run following:    

```
$ helm install stable/nginx-ingress --set controller.hostNetwork=true,rbac.create=true
```


Then install the mocular via following commands:    

```
$ helm repo add monocular https://kubernetes-helm.github.io/monocular
$ helm install monocular/monocular
```
Check the installed packages and its running status:    

```
$ helm ls
NAME             	REVISION	UPDATED                 	STATUS  	CHART               	NAMESPACE
fallacious-jaguar	1       	Thu Jan 11 11:56:21 2018	DEPLOYED	nginx-ingress-0.8.23	default  
incindiary-prawn 	1       	Thu Jan 11 11:58:42 2018	DEPLOYED	monocular-0.5.0     	default  
$ kubectl get pods
NAME                                                              READY     STATUS              RESTARTS   AGE
fallacious-jaguar-nginx-ingress-controller-55cd4578cb-vpn2q       1/1       Running             0          3m
fallacious-jaguar-nginx-ingress-default-backend-5b7d684c6fdzk2m   1/1       Running             0          3m
hello-minikube-7844bdb9c6-596f9                                   1/1       Running             4          11d
incindiary-prawn-mongodb-5d96bdcbc5-47js2                         0/1       ContainerCreating   0          37s
incindiary-prawn-monocular-api-7758c78d8f-j64qx                   0/1       ContainerCreating   0          37s
incindiary-prawn-monocular-api-7758c78d8f-kb7nq                   0/1       ContainerCreating   0          37s
incindiary-prawn-monocular-prerender-65b576dd76-jwvmc             0/1       ContainerCreating   0          37s
incindiary-prawn-monocular-ui-5545f44ffb-7557l                    0/1       ContainerCreating   0          37s
incindiary-prawn-monocular-ui-5545f44ffb-bltmc                    0/1       ContainerCreating   0          37s
```

Get the deployment:    

```
# kubectl get pods --watch
# kubectl get ingress
NAME                         HOSTS     ADDRESS          PORTS     AGE
incindiary-prawn-monocular   *         192.168.99.100   80        2h
# firefox 192.168.99.100
```
Displayed image:   

### Deploy Wordpress
Deploy with following commands:    

```
# helm install --name=wordpress-test1 --set "persistence.enabled=false,mariadb.persistence.enabled=false,serviceType=ClusterIP" stable/wordpress
``` 
Examine the deployment:    

```
# kubectl get pods | grep wordpress
wordpress-test1-mariadb-56c66786cc-2nj8c                          0/1       PodInitializing     0          25s
wordpress-test1-wordpress-6c949bdcb4-22fk4                        0/1       ContainerCreating   0          25s
```

