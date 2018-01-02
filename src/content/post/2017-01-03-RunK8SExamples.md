+++
date = "2017-01-03T19:10:32+08:00"
categories = ["Virtualization"]
keywords = ["Virtualization"]
description = "Run k8s"
title = "运行K8S例程"

+++
### GuestBook
注意修改imagePullPolicy为`IfNotPresent`, 创建服务的步骤分别为:    

```
$ kubectl create -f redis-master-deployment.yaml
$ kubectl create -f redis-master-service.yaml
$ kubectl create -f frontend-deployment.yaml
$ kubectl create -f frontend-service.yaml
```
现在得到其运行状态:    

```
$ kubectl get pod
NAME                            READY     STATUS    RESTARTS   AGE
frontend-88237173-02dvl         1/1       Running   0          2h
frontend-88237173-r7g3v         1/1       Running   0          2h
frontend-88237173-vjbv5         1/1       Running   0          2h
redis-master-4154998525-f186t   1/1       Running   0          2h
redis-slave-132015689-3qh7b     1/1       Running   0          2h
redis-slave-132015689-hpw88     1/1       Running   0          2h
```
可以用proxy-forward直接访问某个pod中暴露出来的frontend服务:    

```
$ kubectl port-forward frontend-88237173-02dvl 9081:80
```
上述命令的意思是，将pod `frontend-88237173-02dvl`80端口的流量转发到
本地的9081端口，则可以通过访问`http://127.0.0.1:9081`来访问frontend.    

或者，我们可以在service文件中指定服务类型为`NodePort`, 定义文件修改如下:    

```
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 31080
```
服务创建以后，访问`http://CoreOS1IP:31080`则可访问到guestbook前端页面，三个CoreOS
节点的31080端口均可提供前端页面访问。    
