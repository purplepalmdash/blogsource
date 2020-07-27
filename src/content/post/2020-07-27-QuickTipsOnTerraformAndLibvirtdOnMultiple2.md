+++
title= "QuickTipsOnTerraformAndLibvirtdOnMultiple2"
date = "2020-07-27T17:03:08+08:00"
description = "QuickTipsOnTerraformAndLibvirtdOnMultiple2"
keywords = ["Technology"]
categories = ["Technology"]
+++
紧接上一篇，这一篇引入`Cluster Mesh`用于暴露全局服务。    

### Cluster Mesh
因为我们上一节搭建的集群中已经设定了`cluster-name`及`cluster-id`， 故不需要对其进行更改，如果没有设置，则需要手动更改:    

```
# kubectl -n kube-system edit cm cilium-config
[ ... add/edit ... ]
cluster-name: cluster1
cluster-id: "1"
```
两个集群上分别创建etcd服务的NodePort服务暴露:    

```
root@mouse-1:/mnt/Rong_cilium# kubectl apply -f etcdNodePort.yaml  -n kube-system
cservice/cilium-etcd-external created
root@mouse-1:/mnt/Rong_cilium# cat etcdNodePort.yaml 
apiVersion: v1
kind: Service
metadata:
  name: cilium-etcd-external
spec:
  type: NodePort
  ports:
  - port: 2379
  selector:
    app: etcd
    etcd_cluster: cilium-etcd
    io.cilium/app: etcd-operator
```
克隆` cilium/clustermesh-tools` 仓库，它含有用于取出密码及生成Kubernetes secrets配置的脚本文件，

```
# git clone https://github.com/cilium/clustermesh-tools.git
# cd clustermesh-tools
# cd clustermesh-tools/
# ls
clustermesh.yaml  config  ds.patch  extract-etcd-secrets.sh  generate-name-mapping.sh  generate-secret-yaml.sh
# rm -rf config/
# rm -f clustermesh.yaml 
# rm -f ds.patch 
```

生成`etcd-secrets`:    

```
root@mouse-1:/mnt/Rong_cilium/clustermesh-tools# ./extract-etcd-secrets.sh 
Derived cluster-name c1 from present ConfigMap
====================================================
 WARNING: The directory config contains private keys.
          Delete after use.
====================================================
```
在另一集群上重复以上操作。   

从上面解压出来的keys/certificates生成单个Kubernetes secret， 这个secret将含有service IP/etcd节点名等用于访问的键/值。   

```
# ./generate-secret-yaml.sh > clustermesh.yaml
```
两个节点上分别运行以下命令，用于生成用于插入到`cilium` daemonset中的所需字段：    

```
# # ./generate-name-mapping.sh > ds.patch
root@cilium-1:/mnt/Rong_cilium_2/clustermesh-tools# ./generate-name-mapping.sh > ds.patch
root@cilium-1:/mnt/Rong_cilium_2/clustermesh-tools# cat ds.patch 
spec:
  template:
    spec:
      hostAliases:
      - ip: "10.137.149.72"
        hostnames:
        - c2.mesh.cilium.io
      - ip: "10.137.149.73"
        hostnames:
        - c2.mesh.cilium.io
root@mouse-1:/mnt/Rong_cilium/clustermesh-tools# ./generate-name-mapping.sh > ds.patch
root@mouse-1:/mnt/Rong_cilium/clustermesh-tools# cat ds.patch 
spec:
  template:
    spec:
      hostAliases:
      - ip: "10.137.149.61"
        hostnames:
        - c1.mesh.cilium.io
      - ip: "10.137.149.62"
        hostnames:
        - c1.mesh.cilium.io
```
组合出一个新的文件:     

```
# cat combine_ds.patch 
spec:
  template:
    spec:
      hostAliases:
      - ip: "10.137.149.61"
        hostnames:
        - c1.mesh.cilium.io
      - ip: "10.137.149.62"
        hostnames:
        - c1.mesh.cilium.io
      - ip: "10.137.149.72"
        hostnames:
        - c2.mesh.cilium.io
      - ip: "10.137.149.73"
        hostnames:
        - c2.mesh.cilium.io
```
两个集群上分别apply：    

```
# kubectl -n kube-system patch ds cilium -p "$(cat combine_ds.patch)"
daemonset.apps/cilium patched
```
两个集群上分别apply `cluster-mesh`:    

```
root@mouse-1:/mnt/Rong_cilium/clustermesh-tools# kubectl -n kube-system apply -f clustermesh.yaml
secret/cilium-clustermesh created
root@mouse-1:/mnt/Rong_cilium/clustermesh-tools# kubectl -n kube-system apply -f /mnt/Rong_cilium_2/clustermesh-tools/clustermesh.yaml 

root@cilium-1:/mnt/Rong_cilium_2/clustermesh-tools# kubectl -n kube-system apply -f clustermesh.yaml
secret/cilium-clustermesh created
root@cilium-1:/mnt/Rong_cilium_2/clustermesh-tools# kubectl -n kube-system apply -f /mnt/Rong_cilium/clustermesh-tools/clustermesh.yaml 
secret/cilium-clustermesh configured
```
重新启动所有节点上的`cilium-agent`，以便使用新的`cilium-clustermesh`密码文件中标识的集群名称、cluster id等，    

```
root@cilium-1:/mnt/Rong_cilium_2/clustermesh-tools# kubectl -n kube-system delete pod -l k8s-app=cilium
pod "cilium-pb7rd" deleted
pod "cilium-wqdrv" deleted
root@cilium-1:/mnt/Rong_cilium_2/clustermesh-tools# kubectl -n kube-system delete pod -l name=cilium-operator
pod "cilium-operator-7cd598bdf6-42lwl" deleted
root@mouse-1:/mnt/Rong_cilium/clustermesh-tools# kubectl -n kube-system delete pod -l k8s-app=cilium
pod "cilium-5k5rj" deleted
pod "cilium-7m4gv" deleted
root@mouse-1:/mnt/Rong_cilium/clustermesh-tools# kubectl -n kube-system delete pod -l name=cilium-operator
pod "cilium-operator-7cd598bdf6-5s4q9" deleted
```
测试集群pod的连接性:    

```
root@cilium-1:/home/test# kubectl exec -ti cilium-sqjm9 -n kube-system cilium node list
Name          IPv4 Address    Endpoint CIDR    IPv6 Address   Endpoint CIDR
c1/mouse-1    10.137.149.61   10.233.64.0/24                  
c1/mouse-2    10.137.149.62   10.233.65.0/24                  
c2/cilium-1   10.137.149.72   10.234.65.0/24                  
c2/cilium-2   10.137.149.73   10.234.64.0/24  
```

```
# kubectl run foo1 -it --rm --image  byrnedo/alpine-curl:latest  --command -- sh -c "curl nginx"
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
# root@mouse-1:/home/test# kubectl logs foo1-78cf747c46-xmk9f
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
如何证明其互通性？可以看到我们创建的curl pod是位于集群1中的，而所有的nginx负载则来自于受控集群(集群2）:    

```
root@mouse-1:/home/test# kubectl get pods -o wide
NAME                     READY   STATUS             RESTARTS   AGE    IP              NODE           NOMINATED NODE   READINESS GATES
foo-76db7d689b-vzlkg     0/1     CrashLoopBackOff   3          95s    10.233.64.84    mouse-1        <none>           <none>
foo1-78cf747c46-xmk9f    0/1     CrashLoopBackOff   3          118s   10.233.64.31    mouse-1        <none>           <none>
nginx-58b97d4885-8cw76   1/1     Running            0          50m    10.234.64.236   admiralty-c2   <none>           <none>
nginx-58b97d4885-c6rkm   1/1     Running            0          50m    10.234.65.98    admiralty-c2   <none>           <none>
nginx-58b97d4885-l2jmj   1/1     Running            0          49m    10.234.64.116   admiralty-c2   <none>           <none>
nginx-58b97d4885-r4p6p   1/1     Running            0          50m    10.234.65.54    admiralty-c2   <none>           <none>
nginx-58b97d4885-sc2jh   1/1     Running            0          49m    10.234.65.139   admiralty-c2   <none>           <none>
nginx-58b97d4885-t5vtx   1/1     Running            0          49m    10.234.64.177   admiralty-c2   <none>           <none>
nginx-58b97d4885-v4lz4   1/1     Running            0          50m    10.234.64.159   admiralty-c2   <none>           <none>
nginx-58b97d4885-wj4xf   1/1     Running            0          50m    10.234.65.82    admiralty-c2   <none>           <none>
nginx-58b97d4885-x4tpv   1/1     Running            0          49m    10.234.65.69    admiralty-c2   <none>           <none>
nginx-58b97d4885-xzfmx   1/1     Running            0          49m    10.234.64.18    admiralty-c2   <none>           <none>
```

