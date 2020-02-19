+++
title = "WorkingTipsOnkubeadmsslTwo"
date = "2020-01-09T11:20:08+08:00"
description = "WorkingTipsOnkubeadmsslTwo"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 补充
重启后，在更新过签名的节点上，验证是否可以获取所有的节点, 并列举出当前可用的更新过后的token:    

```
# kubectl get nodes
# kubectl token list
```
如果没有有效token的话，可以手动创建一个:    

```
# kubectl token create
```
token应该看起来像`6dihyb.d09sbgae8ph2atjw`.      

ssh到每一个工作节点上，重新连接到master节点上:    

```
# kubeadm join --token=<上一布获取的token>  <master节点IP>:<默认为6443端口> --node-name <master节点node名称>
```

重新连接后，可以在master节点上验证是否连接成功：    

```
# kubectl get nodes
```
