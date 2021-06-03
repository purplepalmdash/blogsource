+++
title= "TroubleShootingOnK8snginxproxy"
date = "2020-11-30T14:03:07+08:00"
description = "TroubleShootingOnK8snginxproxy"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 现象
Not Ready:    

```
# kubectl get nodes

ai05 NotReady, SchedulingDisabled node	436d 	v1.13.5
```
### 对策
去掉`SchedulingDisabled`:    

```
# kubectl uncordon ai05
```

`NotReady`的解决方法是:    

```
cd /etc/nginx/
mv nginx.conf nginx.conf_sb
```
某SB改动了此节点上的nginx配置文件，导致该节点无法与正确的api server通信。    
更改为正确的`nginx.conf`配置:    

```
stream {
upstream kube_apiserver {
	least_conn;
	server 192.192.185.97:6443;
}
server {
	listen 127.0.0.1:6443;
	....
}
```
之前是被SB更改为本机的8021端口到8020端口的映射。    

重新启动该节点的kubelet， 但是此时无法正常启动，则:    

```
1. 本机从127.0.0.1:6443切换为192.192.185.97:6443,  通过更改/etc/kubernetes/kubelet.conf。  
2. 本机的kube-proxy从127.0.0.1:6443切换为192.192.185.97:6443, 通过更改configmap
3. 删除本机错误的calico。   
4. calico提示/run/systemd/resolve/resolv.conf无法找到, 手动创建链接文件。
5. 删除calico/kube-proxy等pod，使之自动创建。 
6. 现在nginx-proxy被重新创建，现在开始切换回127.0.0.1:6443 
7. 切换回后，删除calico/kube-proxy/nginx-proxy等 pod
8. 现在一切应该正常。

```
SB的一个误操作，一两个小时就没有了，代价沉重。
