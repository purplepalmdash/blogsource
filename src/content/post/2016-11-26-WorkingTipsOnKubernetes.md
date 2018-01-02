+++
categories = ["Virtualization"]
date = "2016-11-26T20:43:20+08:00"
description = "Working tips on kubernetes"
keywords = ["Virtualization"]
title = "WorkingTipsOnKubernetes"

+++
### 先决条件
CentOS 7.2 1511, Vagrant for kvm.    
关闭selinux, 关闭firewalld, 使用以下命令安装docker最新版:    

```
$ curl -sSL \
http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet |
\ sh -
```
IP地址配置:    

```
master	192.168.0.223
node1	192.168.0.224
```
配置无密码登录，master到master, master到node1.    

```
# ssh-copy-id root@192.168.0.223
# ssh-copy-id root@192.168.0.224
```

### 安装kubernetes
修改配置文件如下:    

```
$ cat kubernetes/cluster/centos/config-default.sh 

# Master配置
export MASTER=${MASTER:-"root@192.168.0.223"}
export MASTER_IP=${MASTER#*@}

# Minion节点配置
export NODES=${NODES:-"root@192.168.0.223 root@192.168.0.224"}

# Cluster中含有的节点数
export NUM_NODES=${NUM_NODES:-2}

# service cluster配置的IP地址范围
export SERVICE_CLUSTER_IP_RANGE=${SERVICE_CLUSTER_IP_RANGE:-"192.168.22.0/24"}

# flannel的overlay网络IP地址范围, 不能和上面定义的SERVICE_CLUSTER_IP_RANGE地址范围冲突
export FLANNEL_NET=${FLANNEL_NET:-"172.20.0.0/16"}

# Docker参数，这里我们开启daocloud加速模式
export DOCKER_OPTS=${DOCKER_OPTS:-"--cluster-store=etcd://$MASTER_IP:2379, --registry-mirror=http://1a653205.m.daocloud.io"}
```

开始部署kubernetes集群:    

```
$ KUBERNETES_PROVIDER=centos ./kube-up.sh
```
安装完毕后，运行以下脚本，重新加载docker的配置后，重启两台机器:    

```
$ sudo systemctl stop docker
$ sudo systemctl daemon-reload
$ sudo systemctl start docker
```
重启完毕后，运行以下命令查看节点状态:    

```
# kubectl get nodes
NAME            STATUS    AGE
192.168.0.223   Ready     10h
192.168.0.224   Ready     10h
```
加载两个docker镜像:    

```
# docker images
gcr.io/google_containers/kubernetes-dashboard-amd64   v1.4.0  
gcr.io/google_containers/pause-amd64                  3.0     
```
添加kubectl到系统路径中:    

```
# cp cluster/centos/binaries/kubectl /opt/kubernetes/bin/
# vim ~/.bashrc
export PATH=$PATH:/opt/kubernetes/bin/
```
### 启动kubernetes UI
启动dashboard-controller服务和dashboard-service服务:    

```
# cd ./cluster/gce/coreos/kube-manifests/addons/dashboard
# kubectl create -f dashboard-controller.yaml 
# kubectl create -f dashboard-service.yaml 
```
在node1上可以查看docker运行状态:    

```
# docker ps
b22047f30d55        gcr.io/google_containers/kubernetes-dashboard-amd64:v1.4.0   "/dashboard --port=90"
8979b7b1db14        gcr.io/google_containers/pause-amd64:3.0                     "/pause"
```

查看`http://192.168.0.223:8080/ui`，得到dashboard的网页.    

![/images/2016_11_26_22_01_29_891x497.jpg](/images/2016_11_26_22_01_29_891x497.jpg)    

service截图:    

![/images/2016_11_26_22_03_57_758x176.jpg](/images/2016_11_26_22_03_57_758x176.jpg)    

### nginx服务
配置nginx-rc.yaml文件用于启动nginx服务:    

```
$ vim nginx-rc.yaml 
apiVersion: v1 
kind: ReplicationController 
metadata: 
  name: nginx-controller 
spec: 
  replicas: 2 
  selector: 
    name: nginx 
  template: 
    metadata: 
      labels: 
        name: nginx 
    spec: 
      containers: 
        - name: nginx
          image: nginx
          ports: 
            - containerPort: 80
```
启动服务:    

```
$ kubectl -s http://192.168.0.223:8080 create -f nginx-rc.yaml
```
检查创建的pods情况:    

```
# kubectl get pods
NAME                     READY     STATUS              RESTARTS   AGE
nginx-controller-1bx6j   0/1       ContainerCreating   0          13s
nginx-controller-attgh   0/1       ContainerCreating   0          13s
```
得到pod的运行情况:    

```
# kubectl describe pod nginx-controller-1bx6j
Name:		nginx-controller-1bx6j
Namespace:	default
Node:		192.168.0.224/192.168.0.224
Start Time:	Sat, 26 Nov 2016 14:08:33 +0000
Labels:		name=nginx
Status:		Running
IP:		172.20.99.3
Controllers:	ReplicationController/nginx-controller
......
```
此刻可以在对应的节点上看到nginx的运行情况: `curl 172.20.99.3`在master节点上。   

### 节点内可访问的nginx service
Service的type有ClusterIP和NodePort之分，缺省是ClusterIP，这种类型的Service
只能在集群内部访问。配置文件如下：    

```
$ vim nginx-service-clusterip.yaml 
apiVersion: v1 
kind: Service 
metadata: 
  name: nginx-service-clusterip 
spec: 
  ports: 
    - port: 8001 
      targetPort: 80 
      protocol: TCP 
  selector: 
    name: nginx
``` 
创建服务:   

```
# kubectl create -f nginx-service-clusterip.yaml
```
查看创建出的service情况:    

```
# kubectl get service
NAME                      CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes                192.168.22.1     <none>        443/TCP    34m
nginx-service-clusterip   192.168.22.189   <none>        8001/TCP   27s
```

访问节点内可访问的nginx service, 在master和node1上都可以访问到:    

```
$ curl 192.168.22.189:8001
```
### 外部可访问的nginx服务
type为NodePort的为外部可访问的nginx服务，定义文件如下:    

```
$ vim nginx-service-nodeport.yaml 
apiVersion: v1 
kind: Service 
metadata: 
  name: nginx-service-nodeport 
spec: 
  ports: 
    - port: 8000
      targetPort: 80 
      protocol: TCP 
  type: NodePort
  selector: 
    name: nginx
```
运行`kubectl create -f nginx-service-nodeport.yaml`, 得到服务如下:    

```
# kubectl get service
NAME                      CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes                192.168.22.1     <none>        443/TCP    58m
nginx-service-clusterip   192.168.22.189   <none>        8001/TCP   23m
nginx-service-nodeport    192.168.22.209   <nodes>       8000/TCP   2m
```
可以看到，新增加了一个`nginx-service-nodeport`的服务.    

查看其端口，30923为其映射的端口号:    

```
# kubectl describe service nginx-service-nodeport
Name:			nginx-service-nodeport
Namespace:		default
Labels:			<none>
Selector:		name=nginx
Type:			NodePort
IP:			192.168.22.209
Port:			<unset>	8000/TCP
NodePort:		<unset>	30923/TCP
Endpoints:		172.20.62.2:80,172.20.99.3:80
Session Affinity:	None
```

从外部(192.168.0.220)验证nginx服务:    
```
# curl http://192.168.0.223:30923
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

### 删除服务
删除我们上面创建的基于nodeport的服务:    

```
# kubectl delete -f ./nginx-service-nodeport.yaml
```
删除以后，检查service情况:    

```
# kubectl get service
NAME                      CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes                192.168.22.1     <none>        443/TCP    1h
nginx-service-clusterip   192.168.22.189   <none>        8001/TCP   55m
```
可以看到`nginx-service-nodeport`服务已经被删除。    

### 指定端口
创建配置文件如下:    

```
$ vim specifynode.yaml 
apiVersion: v1 
kind: Service 
metadata: 
  name: nginx-service-nodeport 
spec: 
  ports: 
    - port: 80
      nodePort: 30080 
      protocol: TCP 
  type: NodePort
  selector: 
    name: nginx
```
创建服务:    

```
$ kubectl create -f ./specifynode.yaml
```
现在访问`http://192.168.0.223:30080`即可访问到nginx服务.    

注: 端口需要绑定在`30000-32767`.    
