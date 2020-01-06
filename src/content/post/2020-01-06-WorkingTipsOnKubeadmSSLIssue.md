+++
title = "WorkingTipsOnKubeadmIssue"
date = "2020-01-06T00:03:39+08:00"
description = "WorkingTipsOnKubeadmIssue"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 环境
IP地址: 10.0.2.15    
主机名: node1    
集群部署于2020年1月6日，kubeadm ssl签名有效期为1年。    
手动调整虚拟机时间为2021年3月3日后，ssl签名失效， kubelet无法启动。    
### 问题
kubeadm ssl签名过期后kubelet进程无法启动：    

```
root@node1:/etc/kubernetes/ssl# systemctl status kubelet
● kubelet.service - Kubernetes Kubelet Server
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
   Active: activating (auto-restart) (Result: exit-code) since Tue 2021-03-02 16:04:46 UTC; 5s ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
  Process: 2001 ExecStart=/usr/local/bin/kubelet $KUBE_LOGTOSTDERR $KUBE_LOG_LEVEL $KUBELET_API_SERVER $KUBELET_ADDRESS $KUBELET_PORT $KUBELET_HOSTNAME $KUBE_
  Process: 1995 ExecStartPre=/bin/mkdir -p /var/lib/kubelet/volume-plugins (code=exited, status=0/SUCCESS)
 Main PID: 2001 (code=exited, status=255)

Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.446479    2001 feature_gate.go:206] feature gates: &{map[]}
Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.466612    2001 mount_linux.go:179] Detected OS with systemd
Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.466684    2001 server.go:408] Version: v1.12.3
Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.466827    2001 feature_gate.go:206] feature gates: &{map[]}
Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.466945    2001 feature_gate.go:206] feature gates: &{map[]}
Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.467156    2001 plugins.go:99] No cloud provider specified.
Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.467192    2001 server.go:524] No cloud provider specified: "" from the config file: ""
Mar 02 16:04:46 node1 kubelet[2001]: E0302 16:04:46.474463    2001 bootstrap.go:205] Part of the existing bootstrap client certificate is expired: 2021-01-05 
Mar 02 16:04:46 node1 kubelet[2001]: I0302 16:04:46.474487    2001 bootstrap.go:61] Using bootstrap kubeconfig to generate TLS client cert, key and kubeconfig
Mar 02 16:04:46 node1 kubelet[2001]: F0302 16:04:46.474525    2001 server.go:262] failed to run Kubelet: unable to load bootstrap kubeconfig: stat /etc/kubern
```
### 解决步骤
创建`pki`目录软链接:     

```
# cd /etc/kubernetes
# ln -s ssl pki
```
备份原有签名(已过期):     

```
#  sudo mv /etc/kubernetes/pki/apiserver.key /etc/kubernetes/pki/apiserver.key.old
#  sudo mv /etc/kubernetes/pki/apiserver.crt /etc/kubernetes/pki/apiserver.crt.old
#  sudo mv /etc/kubernetes/pki/apiserver-kubelet-client.crt /etc/kubernetes/pki/apiserver-kubelet-client.crt.old
#  sudo mv /etc/kubernetes/pki/apiserver-kubelet-client.key /etc/kubernetes/pki/apiserver-kubelet-client.key.old
#  sudo mv /etc/kubernetes/pki/front-proxy-client.crt /etc/kubernetes/pki/front-proxy-client.crt.old
#  sudo mv /etc/kubernetes/pki/front-proxy-client.key /etc/kubernetes/pki/front-proxy-client.key.old
```
创建新的apiserver, apiserver-kubelet-client, front-proxy-client 签名及key值(`10.0.2.15`为K8S主机IP地址):    

```
# sudo kubeadm alpha phase certs apiserver --apiserver-advertise-address 10.0.2.15
# sudo kubeadm alpha phase certs apiserver-kubelet-client
# sudo kubeadm alpha phase certs front-proxy-client
```
备份原有配置文件（已过期）：    

```
# sudo mv /etc/kubernetes/admin.conf /etc/kubernetes/admin.conf.old
# sudo mv /etc/kubernetes/kubelet.conf /etc/kubernetes/kubelet.conf.old
# sudo mv /etc/kubernetes/controller-manager.conf /etc/kubernetes/controller-manager.conf.old
# sudo mv /etc/kubernetes/scheduler.conf /etc/kubernetes/scheduler.conf.old
```
创建新配置文件(注意IP地址和主机名）:    

```
# kubeadm alpha phase kubeconfig all --apiserver-advertise-address 10.0.2.15 --node-name node1
```
更新配置文件，确保kubectl能使用更新后的配置文件:   

```
# su root
#  mv ~/.kube/config  ~/.kube/config.old
#  cp -i /etc/kubernetes/admin.conf ~/.kube/config
#  chmod 777 ~/.kube/config
#  chown root:root /root/.kube/config
```
现在重启整个机器，从而使得新的配置生效。    

### 验证
检查ssl是否被更新(模拟环境中，本地时间为2021年3月2日，而签名档已更新为2022年3月2日前有效):    

```
root@node1:/etc/kubernetes/ssl# for i in `ls *.crt`; do openssl x509 -in $i -noout -dates; done | grep notAfter
notAfter=Mar  2 16:11:50 2022 GMT
notAfter=Mar  2 16:12:11 2022 GMT
notAfter=Jan  3 14:12:03 2030 GMT
notAfter=Jan  3 14:12:04 2030 GMT
notAfter=Mar  2 16:12:20 2022 GMT
root@node1:/etc/kubernetes/ssl# 
root@node1:/etc/kubernetes/ssl# date
Tue Mar  2 16:23:08 UTC 2021
```

检查`kubelet`服务及集群情况（伪造数据，已运行400多天）:    

```
# systemctl status kubelet
● kubelet.service - Kubernetes Kubelet Server
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2021-03-02 16:21:21 UTC; 42s ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
  Process: 1409 ExecStartPre=/bin/mkdir -p /var/lib/kubelet/volume-plugins (code=exited, status=0/SUCCESS)
 Main PID: 1415 (kubelet)
    Tasks: 0
   Memory: 113.4M
      CPU: 1.869s
   CGroup: /system.slice/kubelet.service
           ‣ 1415 /usr/local/bin/kubelet --logtostderr=true --v=2 --address=0.0.0.0 --node-ip=10.0.2.15 --hostname-override=node1 --allow-privileged=true --bo

Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.661 [INFO][2900] ipam.go 1002: Releasing all IPs with handle 'cni0.3f1cc2ef7ca9265d9803d6338d02d08dcc
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.663 [INFO][2900] ipam.go 1021: Querying block so we can release IPs by handle cidr=10.233.102.128/26 
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.668 [INFO][2900] ipam.go 1041: Block has 1 IPs with the given handle cidr=10.233.102.128/26 handle="c
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.668 [INFO][2900] ipam.go 1063: Updating block to release IPs cidr=10.233.102.128/26 handle="cni0.3f1c
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.671 [INFO][2900] ipam.go 1076: Successfully released IPs from block cidr=10.233.102.128/26 handle="cn
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.719 [INFO][2900] calico-ipam.go 267: Released address using handleID ContainerID="3f1cc2ef7ca9265d980
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.719 [INFO][2900] calico-ipam.go 276: Releasing address using workloadID ContainerID="3f1cc2ef7ca9265d
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.720 [INFO][2900] ipam.go 1002: Releasing all IPs with handle 'kube-system.kubernetes-dashboard-5db4d9
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.807 [INFO][2763] k8s.go 358: Cleaning up netns ContainerID="3f1cc2ef7ca9265d9803d6338d02d08dcc2e0e6ac
Mar 02 16:22:02 node1 kubelet[1415]: 2021-03-02 16:22:02.807 [INFO][2763] k8s.go 370: Teardown processing complete. ContainerID="3f1cc2ef7ca9265d9803d6338d02d
root@node1:/home/vagrant# kubectl get nodes
NAME    STATUS   ROLES         AGE    VERSION
node1   Ready    master,node   421d   v1.12.5
```
