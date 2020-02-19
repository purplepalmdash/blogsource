+++
title = "ThinkingOnK8s"
date = "2019-05-21T11:42:55+08:00"
description = "ThinkingOnK8s"
keywords = ["Linux"]
categories = ["Technology"]
+++
上午想了一下关于开发后续要努力的方向。    

### K8S去中心化
早先做的关于Rong的方案都是中心化的，各个组件依赖于一个中心化的kube-deploy节点，当然这个节点也是我对于kubespray项目的一个拓展吧，把dns服务器/secure-registry/harbor服务都落地于一个中心化的节点来做。好处是做到了中心化管理，同时通过欺骗各个节点签名档的方式，无缝衔接了docker.io/gcr.io/quay.io等的pull/push请求。坏处是无法做到高可用，或者说如果做高可用，该如何来设计这个节点呢？    

思路：    

```
1. registry on k8s?   
2. harbor on k8s?   
```

### AI on K8s
有不错的起点，就是Ffdl这个平台。   
但问题是我需要一个去中心化后的节点来做为起步。    

眼下不管是Rong的内网版或者是去中心化的外网版本看来都不是特别理想的实现平台。    

### clearLinux
这个可以作为后续的起点系统，深入研究。    
