+++
title= "WorkingTipsOnKubeadmSSL100YIssue"
date = "2020-12-03T10:37:07+08:00"
description = "WorkingTipsOnKubeadmSSL100YIssue"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Kubernetes v1.15.3
默认已开启100年签名，但节点未开启签名的自动更新，通过以下方法开启:    

```
# cat rr.yml
---
- hosts: k8s-cluster
  gather_facts: false
  tasks:
    - name: "Change kubelet configuration for adding certificates rotate"
      raw: sed -i.$(date "+%m%d%y") '/^clusterDNS:/i rotateCertificates:\ true' /etc/kubernetes/kubelet-config.yaml

- hosts: k8s-cluster
  gather_facts: false
  tasks:
    - name: "Restart kubelet"
      raw: systemctl restart kubelet
# ansible-playbook -i inventory/rong/hosts.ini rr.yml
```
如果已经过期, 则通过rejoin的方式重新加工作节点即可。    
### Kubernetes v1.17.6
Download source file from github:    

```
# wget https://github.com/kubernetes/kubernetes/archive/v1.17.6.zip
# unzip v1.17.6.zip
# cd kubernetes-1.17.6
# vim cmd/kubeadm/app/constants/constants.go
CertificateValidity = time.Hour * 24 * 365 * 100
#  vim vendor/k8s.io/client-go/util/cert/cert.go
func NewSelfSignedCACert
NotAfter: 	now.Add(duration365d * 100).UTC(),
func GenerateSelfSignedCertKeyWithFixtures
maxAge := 100 * time.Hour * 24 * 365
```
Edit building:    

```
# vim hack/make-rules/cross.sh
    make all WHAT="${KUBE_SERVER_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_SERVER_PLATFORMS[*]}"
    
    #make all WHAT="${KUBE_NODE_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_NODE_PLATFORMS[*]}"
    #
    #make all WHAT="${KUBE_CLIENT_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_CLIENT_PLATFORMS[*]}"
    #
    #make all WHAT="${KUBE_TEST_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_TEST_PLATFORMS[*]}"
    #
    #make all WHAT="${KUBE_TEST_SERVER_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_TEST_SERVER_PLATFORMS[*]}"
#  vim hack/lib/golang.sh
    readonly KUBE_SUPPORTED_SERVER_PLATFORMS=(
    #  linux/amd64
    #  linux/arm
      linux/arm64
    #  linux/s390x
    #  linux/ppc64le
    )
    
    //.............
    
    kube::golang::server_targets() {
      local targets=(                       
      #  cmd/kube-proxy                                                        
      #  cmd/kube-apiserver                 
      #  cmd/kube-controller-manager                                 
      #  cmd/kubelet 
        cmd/kubeadm
      #  cmd/kube-scheduler
      #  vendor/k8s.io/apiextensions-apiserver
      #  cluster/gce/gci/mounter
      )
```
Build:    

```
# make cross
```

Get the build result:    

```
# ls ./_output/local/go/bin/linux_arm64/kubeadm
# file ./_output/local/go/bin/linux_arm64/kubeadm
./_output/local/go/bin/linux_arm64/kubeadm: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, stripped
```
### Kubernetes v1.18.8(arm64)
The same as in v1.17.6

### Update(V1.17.6)
默认情况下集群状态(365天过期):    

```
root@wuhanarm64-1:/home/test# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
W1203 11:44:01.023274   43160 defaults.go:186] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [110.192.0.10]; the provided value is: [110.192.0.3]

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Sep 11, 2021 06:45 UTC   282d                                    no      
apiserver                  Sep 11, 2021 06:43 UTC   282d            ca                      no      
apiserver-kubelet-client   Sep 11, 2021 06:43 UTC   282d            ca                      no      
controller-manager.conf    Sep 11, 2021 06:45 UTC   282d                                    no      
front-proxy-client         Sep 11, 2021 06:43 UTC   282d            front-proxy-ca          no      
scheduler.conf             Sep 11, 2021 06:45 UTC   282d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Sep 09, 2030 06:43 UTC   9y              no      
front-proxy-ca          Sep 09, 2030 06:43 UTC   9y              no      
root@wuhanarm64-1:/home/test# kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
wuhanarm64-1   Ready    master   82d   v1.17.6
wuhanarm64-2   Ready    <none>   82d   v1.17.6
wuhanarm64-3   Ready    <none>   82d   v1.17.6
```
更新kubeadm:    

```
# mv /usr/local/bin/kubeadm /usr/local/bin/kubeadm.back
# scp gowuegowoguweog:gowugouwoeogo/kubeadm_1.17.6_arm64 /usr/local/bin/kubeadm
# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.6", GitCommit:"d32e40e20d167e103faf894261614c5b45c44198", GitTreeState:"archive", BuildDate:"2020-12-03T02:53:08Z", GoVersion:"go1.14.2", Compiler:"gc", Platform:"linux/arm64"}

```
用新生成的kubeadm重新`renew`签名:    

```
 # kubeadm alpha certs renew all --config=/etc/kubernetes/kubeadm-config.yaml

W1203 11:48:50.884660   45705 defaults.go:186] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [110.192.0.10]; the provided value is: [110.192.0.3]
W1203 11:48:50.885133   45705 validation.go:28] Cannot validate kube-proxy config - no validator is available
W1203 11:48:50.885156   45705 validation.go:28] Cannot validate kubelet config - no validator is available
certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
W1203 11:49:55.769736   46233 defaults.go:186] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [110.192.0.10]; the provided value is: [110.192.0.3]

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Nov 09, 2120 03:48 UTC   99y                                     no      
apiserver                  Nov 09, 2120 03:48 UTC   99y             ca                      no      
apiserver-kubelet-client   Nov 09, 2120 03:48 UTC   99y             ca                      no      
controller-manager.conf    Nov 09, 2120 03:48 UTC   99y                                     no      
front-proxy-client         Nov 09, 2120 03:48 UTC   99y             front-proxy-ca          no      
scheduler.conf             Nov 09, 2120 03:48 UTC   99y                                     no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Sep 09, 2030 06:43 UTC   9y              no      
front-proxy-ca          Sep 09, 2030 06:43 UTC   9y              no      

```

更新kubeconfig:    

```
# kubeadm init phase kubeconfig all --config kubeadm-config.yaml 
W1203 11:55:20.868908   49000 defaults.go:186] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [110.192.0.10]; the provided value is: [110.192.0.3]
W1203 11:55:20.869702   49000 validation.go:28] Cannot validate kube-proxy config - no validator is available
W1203 11:55:20.869730   49000 validation.go:28] Cannot validate kubelet config - no validator is available
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/admin.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/scheduler.conf"
# mv $HOME/.kube/config $HOME/.kube/config.old
# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# chown $(id -u):$(id -g) $HOME/.kube/config
```
重启 kube-apiserver、kube-controller、kube-scheduler、etcd 这4个容器, 举`kube-apiserver`为例:    

```
# docker ps | grep kube-api
d086ef6a4b9b        6388c24eab51                        "kube-apiserver --ad…"   2 hours ago         Up 2 hours                              k8s_kube-apiserver_kube-apiserver-wuhanarm64-1_kube-system_40f4f19c75fe95ea92ebe00b7bc1576e_625
486a3c2d3e9e        k8s.gcr.io/pause:3.1                "/pause"                 2 hours ago         Up 2 hours                              k8s_POD_kube-apiserver-wuhanarm64-1_kube-system_40f4f19c75fe95ea92ebe00b7bc1576e_3
# docker rm -f d086ef6a4b9b
d086ef6a4b9b
```

检查apiserver证书:    

```
# echo | openssl s_client -showcerts -connect 127.0.0.1:6443 -servername api 2>/dev/null | openssl x509 -noout -enddate
notAfter=Nov  9 03:48:52 2120 GMT
```
1.17.6中，因kubelet默认开启了`rotateCertificates`模式，各节点证书在一年后应该会自动更新。   

### 1.17.6(已过期)
如果已经过期的话，如何处理?    

首先将更改了100年签名的`kubeadm`拷贝到kube-master[0]节点上，替换掉默认的`kubeadm`:    

```
# mv /usr/local/bin/kubeadm /usr/local/bin/kubeadm.back
# scp gowuegowoguweog:gowugouwoeogo/kubeadm_1.17.6_arm64 /usr/local/bin/kubeadm
```
 
使用以下命令查看签名的时间:    

```
# cd /etc/kubernetes/ssl
# for i in `ls *.crt`; do openssl x509 -in $i -noout -dates; done | grep notAfte
notAfter=Oct 31 06:11:04 2020 GMT
notAfter=Oct 31 06:11:03 2020 GMT
notAfter=Oct 31 06:11:03 2020 GMT
notAfter=Oct 31 06:11:04 2020 GMT
notAfter=Oct 31 06:11:05 2020 GMT
```
在kube-master[0]节点上，手动设置时间为过期前的时间，如2020年9月1日:    

```
# date -s 20200901
# hwclock -w
```
在kube-master[0]节点上的`Rong`安装目录里，通过ssh同步所有节点(举例为`10.137.149.231~233`)时间:    

```
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.231 date -s @`( date -u +"%s" )`
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.232 date -s @`( date -u +"%s" )`
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.233 date -s @`( date -u +"%s" )`
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.231 hwclock -w 
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.232 hwclock -w 
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.233 hwclock -w 
```
更改完时间后，`kubectl get node`等命令可用, 此时使用以下命令更新签名:     

```
# kubeadm alpha certs renew all=kubeadm --config=kubeadm-config.yaml
```
因签名已变化，替换掉当前使用的`.kube/config`文件:    

```
# mv $HOME/.kube/config $HOME/.kube/config.old
# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

使用以下命令查看更新后的签名(100年以后过期):    

```
# kubeadm alpha certs check-expiration
```
检查各节点是否为`Ready`状态，如果未ready，可登录至该节点， 通过`systemctl restart kubelet`命令重启控制平面同步签名后即可变成ready状态:   

```
# kubectl get node
```
调整回正确的时间（当前时间）:     

```
# date -s 20201203
# date -s 14:20:20
# hwclock -w
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.231 date -s @`( date -u +"%s" )`
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.232 date -s @`( date -u +"%s" )`
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.233 date -s @`( date -u +"%s" )`
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.231 hwclock -w 
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.232 hwclock -w 
# ssh -o "StrictHostKeyChecking=no" -i .rong/deploy.key root@10.137.149.233 hwclock -w 
```
调整完时间后，再次检查签名是否正常, 各工作节点是否ready:    

```
# kubeadm alpha certs check-expiration
# kubectl get node
```
### v1.18.8(未过期)
将kubeadm(100年修改版)上传到master机器上，renew签名即可。具体步骤与v1.17.6相同。
### v1.18.8(已过期)
具体步骤与v1.17.6(已过期)相同。
