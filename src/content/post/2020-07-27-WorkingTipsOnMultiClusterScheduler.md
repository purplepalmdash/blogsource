+++
title= "WorkingTipsOnMultiClusterScheduler"
date = "2020-07-27T14:22:28+08:00"
description = "WorkingTipsOnMultiClusterScheduler"
keywords = ["Technology"]
categories = ["Technology"]
+++
前段时间有同事给过来一篇很耸人听闻的文章, 号称能将K8S能力扩充到无限：   

![/images/2020_07_27_14_24_35_855x675.jpg](/images/2020_07_27_14_24_35_855x675.jpg)

搭建的过程其实蛮繁琐，最终我也没有将这个方案付诸实际。怎么说呢，感觉这个项目的文档非常不规范，缺乏实际落地的可行性。倒是在研究`virtual-kubelet`的过程中发现了`admiraltyio
/multicluster-scheduler`的解决方案值得一试。搭建过程中也走了不少弯路。以下就是一个从0开始的部署过程，记下来以便以后参考。

### 环境说明
采用双集群方式部署，每个集群由两台机器组成，分别列表展示如下:    

集群1：    

```
主机名/IP/网关
mouse-1 10.137.149.61 10.137.149.1
mouse-2 10.137.149.62 10.137.149.1
kube_pods_subnet: 10.233.64.0/18
kube_service_addresses: 10.233.0.0/18
```

集群2：    

```
主机名/IP/网关
cilium-1 10.137.149.72 10.137.149.1
cilium-2 10.137.149.73 10.137.149.1
kube_service_addresses: 10.234.0.0/18
kube_pods_subnet: 10.234.64.0/18
```
采用kubespray的方法来部署集群，两个集群的部署时间因为是虚拟机的缘故大约在40分钟左右。kubespray默认提供了网络组件的安装的，但这里我们并不需要，注释掉了相关的部署脚本，在后面我们将采用手动的方式来部署cilium作为网络组件。同时因为`cert-manager`认证的原因我们加入了以下标记作为部署时参数:    

```
kube_network_plugin: cilium
kube_apiserver_enable_admission_plugins:
  - "NamespaceLifecycle"
  - "LimitRanger"
  - "ServiceAccount"
  - "DefaultStorageClass"
  - "DefaultTolerationSeconds"
  - "MutatingAdmissionWebhook"
  - "ValidatingAdmissionWebhook"
  - "Priority"
  - "ResourceQuota"
additional_no_proxy: ".domain,corp,.company.com,.svc,.svc.cluster.local"
```
`cluster.yml`中去掉的关于网络组件部署的条目:    

```
#    - { role: kubernetes-apps/network_plugin, tags: network }
....
#- hosts: k8s-cluster
#  gather_facts: False
#  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
#  roles:
#    - { role: kubespray-defaults }
#    - { role: kubernetes/preinstall, when: "dns_mode != 'none' and resolvconf_mode == 'host_resolvconf'", tags: resolvconf, dns_late: true }
```
部署完成后检查节点:    

```
集群1: 
root@mouse-1:/mnt/Rong_cilium# kubectl get nodes
NAME      STATUS     ROLES    AGE     VERSION
mouse-1   NotReady   master   4m47s   v1.17.5
mouse-2   NotReady   <none>   2m16s   v1.17.5
root@mouse-1:/mnt/Rong_cilium# kubectl get pods --all-namespaces
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   coredns-76798d84dd-tl8vb                      0/1     Pending   0          96s
kube-system   dns-autoscaler-85f898cd5c-lsxj5               0/1     Pending   0          85s
kube-system   kube-apiserver-mouse-1                        1/1     Running   0          4m11s
kube-system   kube-controller-manager-mouse-1               1/1     Running   0          4m48s
kube-system   kube-proxy-5jbh7                              1/1     Running   0          2m15s
kube-system   kube-proxy-x75k7                              1/1     Running   0          2m10s
kube-system   kube-scheduler-mouse-1                        1/1     Running   0          4m48s
kube-system   kubernetes-dashboard-857df7d6f7-nrrcj         0/1     Pending   0          69s
kube-system   kubernetes-metrics-scraper-747b4fd5cd-whjpj   0/1     Pending   0          61s
kube-system   metrics-server-754db6c5f7-8hbd4               0/2     Pending   0          31s
kube-system   nginx-proxy-mouse-2                           1/1     Running   1          2m19s

集群2:   
root@cilium-1:/mnt/Rong_cilium_2# kubectl get nodes
NAME       STATUS     ROLES    AGE     VERSION
cilium-1   NotReady   master   4m43s   v1.17.5
cilium-2   NotReady   <none>   2m46s   v1.17.5
root@cilium-1:/mnt/Rong_cilium_2# kubectl get pods --all-namespaces
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   coredns-76798d84dd-7j5xx                      0/1     Pending   0          2m15s
kube-system   dns-autoscaler-85f898cd5c-grrpc               0/1     Pending   0          2m10s
kube-system   kube-apiserver-cilium-1                       1/1     Running   0          4m24s
kube-system   kube-controller-manager-cilium-1              1/1     Running   0          4m24s
kube-system   kube-proxy-l2slf                              1/1     Running   0          2m52s
kube-system   kube-proxy-zf4l2                              1/1     Running   0          2m50s
kube-system   kube-scheduler-cilium-1                       1/1     Running   0          4m24s
kube-system   kubernetes-dashboard-857df7d6f7-sz72b         0/1     Pending   0          2m8s
kube-system   kubernetes-metrics-scraper-747b4fd5cd-t7cmp   0/1     Pending   0          2m8s
kube-system   metrics-server-754db6c5f7-d9bzv               0/2     Pending   0          107s
kube-system   nginx-proxy-cilium-2                          1/1     Running   0          2m55s
root@cilium-1:/mnt/Rong_cilium_2# helm version
version.BuildInfo{Version:"v3.1.2", GitCommit:"d878d4d45863e42fd5cff6743294a11d28a9abce", GitTreeState:"clean", GoVersion:"go1.13.8"}
root@cilium-1:/mnt/Rong_cilium_2# uname -a
Linux cilium-1 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
root@cilium-1:/mnt/Rong_cilium_2# cat /etc/issue
Ubuntu 18.04.4 LTS \n \l
```

### cilium创建
离线状况下，分别在两个集群中运行以下命令以配置cilium：    

集群1: 

```
root@mouse-1:/mnt/Rong_cilium/cilium# helm install cilium . --namespace kube-system   --set global.etcd.enabled=true --set global.etcd.managed=true --set global.ipam.operator.clusterPoolIPv4PodCIDR=10.233.64.0/18 --set global.cluster.id=1  --set global.cluster.name=c1
LAST DEPLOYED: Mon Jul 27 14:54:44 2020
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
You have successfully installed Cilium.

Your release version is 1.8.1.

For any further help, visit https://docs.cilium.io/en/v1.8/gettinghelp

```

集群2:     

```
root@cilium-1:/mnt/Rong_cilium_2/cilium# helm install cilium . --namespace kube-system   --set global.etcd.enabled=true --set global.etcd.managed=true --set global.ipam.operator.clusterPoolIPv4PodCIDR=10.234.64.0/18 --set global.cluster.id=2  --set global.cluster.name=c2
```
如果是在线状态下，则改用以下命令(.......后续命令一样):    

```
$ helm install cilium cilium/cilium --version 1.8.2 --namespace kube-system .........
```

两个节点上均激活CoreDNS的`reverse lookup`:     

```
# kubectl -n kube-system edit cm coredns
[...]
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
    }
```

需要经过大约5～10分钟（视机器性能而异)， 等待所有的服务正常, 以集群1为例:     

```
root@mouse-1:/mnt/Rong_cilium/cilium# kubectl get pods --all-namespaces
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   cilium-6txnn                                  1/1     Running   0          10m
kube-system   cilium-8ltpd                                  1/1     Running   0          10m
kube-system   cilium-etcd-dggrz7wn94                        1/1     Running   0          12m
kube-system   cilium-etcd-j7tzh2jcjg                        1/1     Running   0          7m55s
kube-system   cilium-etcd-operator-584788b99c-z5fzp         1/1     Running   0          19m
kube-system   cilium-etcd-rf5vjsrp7n                        1/1     Running   0          13m
kube-system   cilium-operator-7cd598bdf6-5s4q9              1/1     Running   3          19m
kube-system   coredns-76798d84dd-8p7q5                      1/1     Running   0          15m
kube-system   coredns-76798d84dd-tl8vb                      1/1     Running   0          25m
kube-system   dns-autoscaler-85f898cd5c-lsxj5               1/1     Running   0          25m
kube-system   etcd-operator-59cf4cfb7c-p444s                1/1     Running   0          13m
kube-system   kube-apiserver-mouse-1                        1/1     Running   0          28m
kube-system   kube-controller-manager-mouse-1               1/1     Running   0          28m
kube-system   kube-proxy-5jbh7                              1/1     Running   0          26m
kube-system   kube-proxy-x75k7                              1/1     Running   0          26m
kube-system   kube-scheduler-mouse-1                        1/1     Running   0          28m
kube-system   kubernetes-dashboard-857df7d6f7-nrrcj         1/1     Running   0          25m
kube-system   kubernetes-metrics-scraper-747b4fd5cd-whjpj   1/1     Running   0          24m
kube-system   metrics-server-5c55b76c7-p9j47                2/2     Running   1          15m
kube-system   nginx-proxy-mouse-2                           1/1     Running   1          26m
root@mouse-1:/mnt/Rong_cilium/cilium# kubectl get svc --all-namespaces
NAMESPACE     NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes                  ClusterIP   10.233.0.1      <none>        443/TCP                  28m
kube-system   cilium-etcd                 ClusterIP   None            <none>        2379/TCP,2380/TCP        13m
kube-system   cilium-etcd-client          ClusterIP   10.233.44.240   <none>        2379/TCP                 13m
kube-system   coredns                     ClusterIP   10.233.0.3      <none>        53/UDP,53/TCP,9153/TCP   25m
kube-system   dashboard-metrics-scraper   ClusterIP   10.233.61.59    <none>        8000/TCP                 25m
kube-system   kubernetes-dashboard        ClusterIP   10.233.39.108   <none>        443/TCP                  25m
kube-system   metrics-server              ClusterIP   10.233.41.242   <none>        443/TCP                  24m
```
编辑cilium的ds属性，以使其不调度到将来创建的虚拟节点上:    

```
# kubectl edit ds cilium -n kube-system
spec:
  template:
    spec:
      tolerations:
      - operator: Exists
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: virtual-kubelet.io/provider
                operator: DoesNotExist
```
### multicluster-scheduler
这里我们部署的版本是`v0.10.0-rc.1`, 部署步骤如下:    

在两个集群中分别按以下命令部署cert-manager:    

```
# kubectl apply --validate=false -f  https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml
# kubectl create namespace cert-manager
# helm install cert-manager \
    --namespace cert-manager \
    --version v0.12.0 \
    --wait \
    jetstack/cert-manager
```
注:离线情况下命令如下:    

```
切换到项目根目录
# kubectl apply --validate=false -f 00-crds.yaml
# kubectl create namespace cert-manager
# cd cert-manager/
# helm install cert-manager . --namespace cert-manager
```
分别检查`cert-manager`安装情况:    

```
root@cilium-1:/mnt/Rong_cilium_2/cert-manager# kubectl get pods -n cert-manager
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-754d9b75d9-4nn96             1/1     Running   0          97s
cert-manager-cainjector-85fbdf788-lbhbp   1/1     Running   0          97s
cert-manager-webhook-76f9b64b45-wv2dv     1/1     Running   0          97s
```
### 受控集群
受控集群安装:    

```
# kubectl create ns admiralty
namespace/admiralty created
# helm install multicluster-scheduler admiralty/multicluster-scheduler \
  --namespace admiralty \
  --version 0.10.0-rc.1 \
  --set clusterName=c2
```
离线情况下:    

```
root@cilium-1:/mnt/Rong_cilium_2/multicluster-scheduler# kubectl create ns admiralty
namespace/admiralty created
root@cilium-1:/mnt/Rong_cilium_2/multicluster-scheduler# helm install multicluster-scheduler  . --namespace admiralty --set clusterName=c2
NAME: multicluster-scheduler
LAST DEPLOYED: Mon Jul 27 15:36:12 2020
NAMESPACE: admiralty
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
检查运行情况:    

```
root@cilium-1:/mnt/Rong_cilium_2/multicluster-scheduler# kubectl get pods  -n admiralty
NAME                                                          READY   STATUS    RESTARTS   AGE
multicluster-scheduler-candidate-scheduler-64bf58cbc8-v5mks   1/1     Running   0          7m38s
multicluster-scheduler-controller-manager-5844784fb8-bxpbx    1/1     Running   0          7m38s
multicluster-scheduler-proxy-scheduler-65d58b4548-k7hkb       1/1     Running   0          7m38s
```
### 控制集群
控制集群，即主集群安装:    

```
# kubectl create ns admiralty
namespace/admiralty created
# helm install multicluster-scheduler admiralty/multicluster-scheduler \
  --namespace admiralty \
  --version 0.10.0-rc.1 \
  --set clusterName=c1 \
  --set targetSelf=true \
  --set targets[0].name=c2
```
离线情况下:     

```
root@mouse-1:/mnt/Rong_cilium# kubectl  create namespace admiralty
namespace/admiralty created
root@mouse-1:/mnt/Rong_cilium/multicluster-scheduler# helm install multicluster-scheduler . --namespace admiralty  --set clusterName=c1 --set targetSelf=true --set targets[0].name=c2
NAME: multicluster-scheduler
LAST DEPLOYED: Mon Jul 27 15:41:38 2020
NAMESPACE: admiralty
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
此时检查运行情况:    

```
root@mouse-1:/mnt/Rong_cilium/multicluster-scheduler# kubectl get pods -n admiralty
NAME                                                         READY   STATUS              RESTARTS   AGE
multicluster-scheduler-candidate-scheduler-bc484d49d-dkfjq   1/1     Running             0          2m47s
multicluster-scheduler-controller-manager-7d84b5dc69-2l2nx   0/1     ContainerCreating   0          2m47s
multicluster-scheduler-proxy-scheduler-bf9c844df-6pdcq       0/1     ContainerCreating   0          2m47s
```
`controller-manager`和`proxy-scheduler` pod一直处于`ContainerCreating`状态是正确的，因为在控制集群内我们还没有引入受控集群的secret。   
### 交换Service Account
`受控集群`下，运行以下命令:    

```
# kubectl apply -f https://raw.githubusercontent.com/ibuildthecloud/klum/v0.0.1/deploy.yaml
# cat <<EOF | kubectl --context "$CLUSTER2" apply -f -
kind: User
apiVersion: klum.cattle.io/v1alpha1
metadata:
  name: c1
spec:
  clusterRoles:
    - multicluster-scheduler-source
    - multicluster-scheduler-cluster-summary-viewer
EOF
```
离线情况下，部署的命令为:    

```
# kubectl apply -f deploy-klum.yaml 
namespace/klum created
deployment.apps/klum created
serviceaccount/klum created
clusterrolebinding.rbac.authorization.k8s.io/klum-cluster-admin created
root@cilium-1:/mnt/Rong_cilium_2# cat apply-klum.yaml 
kind: User
apiVersion: klum.cattle.io/v1alpha1
metadata:
  name: c1
spec:
  clusterRoles:
    - multicluster-scheduler-source
    - multicluster-scheduler-cluster-summary-viewer
root@cilium-1:/mnt/Rong_cilium_2# kubectl apply -f apply-klum.yaml 
user.klum.cattle.io/c1 created
```


检查运行情况:     

```
root@cilium-1:/mnt/Rong_cilium_2# kubectl get pods -n klum
NAME                    READY   STATUS    RESTARTS   AGE
klum-74f54765d4-zqv4b   1/1     Running   0          92s
```
下载`kubemcsa`二进制文件，以便于快速导出kubeconfig secret文件:     

```
# MCSA_RELEASE_URL=https://github.com/admiraltyio/multicluster-service-account/releases/download/v0.6.1
# OS=linux # or darwin (i.e., OS X) or windows
# ARCH=amd64 # if you're on a different platform, you must know how to build from source
# curl -Lo kubemcsa "$MCSA_RELEASE_URL/kubemcsa-$OS-$ARCH"
# chmod +x kubemcsa
```
运行`kubemcsa export`生成对应的密码文件:     

```
 ./kubemcsa-linux-amd64 export -n klum c1 --as c2>c1asc2.yaml
```
传输`c1asc2.yaml`文件至` 控制节点 `上并apply， 此时上节中处于`Creating`状态的pod会正常运行:    


```
root@mouse-1:/mnt/Rong_cilium# kubectl -n admiralty apply -f /mnt/Rong_cilium_2/c1asc2.yaml
secret/c2 created
root@mouse-1:/mnt/Rong_cilium# kubectl get pods -n admiralty
NAME                                                         READY   STATUS    RESTARTS   AGE
multicluster-scheduler-candidate-scheduler-bc484d49d-dkfjq   1/1     Running   0          12m
multicluster-scheduler-controller-manager-7d84b5dc69-qc98k   1/1     Running   0          49s
multicluster-scheduler-proxy-scheduler-bf9c844df-fg5hk       1/1     Running   0          39s
```
此时在控制集群的`kube-master`节点上刷新集群节点信息可以看到虚拟节点已经被加入到集群:     

```
root@mouse-1:/mnt/Rong_cilium# kubectl get nodes -o wide
NAME           STATUS   ROLES     AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
admiralty-c1   Ready    cluster   67s             <none>          <none>        <unknown>            <unknown>           <unknown>
admiralty-c2   Ready    cluster   67s             <none>          <none>        <unknown>            <unknown>           <unknown>
mouse-1        Ready    master    69m   v1.17.5   10.137.149.61   <none>        Ubuntu 18.04.4 LTS   4.15.0-76-generic   docker://19.3.9
mouse-2        Ready    <none>    66m   v1.17.5   10.137.149.62   <none>        Ubuntu 18.04.4 LTS   4.15.0-76-generic   docker://19.3.9
```

### 控制集群节点上的Multi-Cluster部署
Multicluster-scheduler的pod admission controller会查看标注为`multicluster-scheduler=enabled`的命名空间，在控制集群的`kube-master`节点上，标记`default`空间:    

```
root@mouse-1:/mnt/Rong_cilium# kubectl label namespace default multicluster-scheduler=enabled
namespace/default labeled
```
接着创建一个NGINX应用，注意中间使用了`election`标记:    

```
root@mouse-1:/mnt/Rong_cilium# cat deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        multicluster.admiralty.io/elect: ""
    spec:
      containers:
      - name: nginx
        image: nginx:1.17
        resources:
          requests:
            cpu: 100m
            memory: 32Mi
        ports:
        - containerPort: 80
root@mouse-1:/mnt/Rong_cilium# kubectl apply -f deploy.yaml 
deployment.apps/nginx created
```

检查运行情况:    

```
root@cilium-1:/home/test# kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
nginx-7bb89ffbfb-bs45b-l468n   1/1     Running   0          89s
nginx-7bb89ffbfb-h6btc-4587q   1/1     Running   0          2m
nginx-7bb89ffbfb-mjpf6-n2gc7   1/1     Running   0          2m3s
nginx-7bb89ffbfb-p77d4-phf9f   1/1     Running   0          82s
nginx-7bb89ffbfb-pbj69-hrjqs   1/1     Running   0          117s


root@mouse-1:/mnt/Rong_cilium# kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE    IP              NODE           NOMINATED NODE   READINESS GATES
nginx-7bb89ffbfb-55px2         1/1     Running   0          101s   10.233.64.91    admiralty-c1   <none>           <none>
nginx-7bb89ffbfb-55px2-dglzq   1/1     Running   0          88s    10.233.64.91    mouse-1        <none>           <none>
nginx-7bb89ffbfb-59tl5         1/1     Running   0          101s   10.233.64.68    admiralty-c1   <none>           <none>
nginx-7bb89ffbfb-59tl5-mjl5b   1/1     Running   0          92s    10.233.64.68    mouse-1        <none>           <none>
nginx-7bb89ffbfb-bs45b         1/1     Running   0          101s   10.234.64.214   admiralty-c2   <none>           <none>
nginx-7bb89ffbfb-dj27d         1/1     Running   0          101s   10.233.64.55    admiralty-c1   <none>           <none>
nginx-7bb89ffbfb-dj27d-8tfwn   1/1     Running   0          53s    10.233.64.55    mouse-1        <none>           <none>
nginx-7bb89ffbfb-h6btc         1/1     Running   0          101s   10.234.64.199   admiralty-c2   <none>           <none>
nginx-7bb89ffbfb-hx7fc         1/1     Running   0          101s   10.233.64.159   admiralty-c1   <none>           <none>
nginx-7bb89ffbfb-hx7fc-tbgcr   1/1     Running   0          84s    10.233.64.159   mouse-1        <none>           <none>
nginx-7bb89ffbfb-lgtmw         1/1     Running   0          101s   10.233.64.223   admiralty-c1   <none>           <none>
nginx-7bb89ffbfb-lgtmw-hp4ww   1/1     Running   0          74s    10.233.64.223   mouse-1        <none>           <none>
nginx-7bb89ffbfb-mjpf6         1/1     Running   0          102s   10.234.64.202   admiralty-c2   <none>           <none>
nginx-7bb89ffbfb-p77d4         1/1     Running   0          101s   10.234.64.110   admiralty-c2   <none>           <none>
nginx-7bb89ffbfb-pbj69         1/1     Running   0          101s   10.234.64.103   admiralty-c2   <none>           <none>
```

加入一个`region` label 用于调度:    

```
控制集群
# kubectl label nodes -l virtual-kubelet.io/provider!=admiralty topology.kubernetes.io/region=us
受控集群
# kubectl label nodes -l virtual-kubelet.io/provider!=admiralty topology.kubernetes.io/region=eu
```

调度到`c2`集群上，因c2集群的`region` 为 `eu`:    

```
root@mouse-1:/mnt/Rong_cilium# kubectl patch deployment nginx -p '{
  "spec":{
    "template":{
      "spec": {
        "nodeSelector": {
          "topology.kubernetes.io/region": "eu"
        }
      }
    }
  }
}'
```
检查调度情况:     

```
root@cilium-1:/mnt/Rong_cilium_2# kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
nginx-58b97d4885-8cw76-65zz7   1/1     Running   0          94s
nginx-58b97d4885-c6rkm-dns4r   1/1     Running   0          112s
nginx-58b97d4885-l2jmj-5hlvf   1/1     Running   0          60s
nginx-58b97d4885-r4p6p-wj98b   1/1     Running   0          110s
nginx-58b97d4885-sc2jh-89d6t   1/1     Running   0          82s
nginx-58b97d4885-t5vtx-6n2nn   1/1     Running   0          46s
nginx-58b97d4885-v4lz4-qfctr   1/1     Running   0          101s
nginx-58b97d4885-wj4xf-hjx6v   1/1     Running   0          89s
nginx-58b97d4885-x4tpv-shvrh   1/1     Running   0          68s
nginx-58b97d4885-xzfmx-vmz8h   1/1     Running   0          34s

root@mouse-1:/mnt/Rong_cilium# kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP              NODE           NOMINATED NODE   READINESS GATES
nginx-58b97d4885-8cw76   1/1     Running   0          103s   10.234.64.236   admiralty-c2   <none>           <none>
nginx-58b97d4885-c6rkm   1/1     Running   0          104s   10.234.65.98    admiralty-c2   <none>           <none>
nginx-58b97d4885-l2jmj   1/1     Running   0          54s    10.234.64.116   admiralty-c2   <none>           <none>
nginx-58b97d4885-r4p6p   1/1     Running   0          104s   10.234.65.54    admiralty-c2   <none>           <none>
nginx-58b97d4885-sc2jh   1/1     Running   0          79s    10.234.65.139   admiralty-c2   <none>           <none>
nginx-58b97d4885-t5vtx   1/1     Running   0          39s    10.234.64.177   admiralty-c2   <none>           <none>
nginx-58b97d4885-v4lz4   1/1     Running   0          104s   10.234.64.159   admiralty-c2   <none>           <none>
nginx-58b97d4885-wj4xf   1/1     Running   0          103s   10.234.65.82    admiralty-c2   <none>           <none>
nginx-58b97d4885-x4tpv   1/1     Running   0          61s    10.234.65.69    admiralty-c2   <none>           <none>
nginx-58b97d4885-xzfmx   1/1     Running   0          35s    10.234.64.18    admiralty-c2   <none>           <none>
```
由上可以看到，虚拟节点及c2物理节点上存在一一对应, 而更改为us则全部会切换回控制集群。    
