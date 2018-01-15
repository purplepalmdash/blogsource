+++
title = "TroubleShootingOnhelm"
date = "2018-01-12T15:22:33+08:00"
description = "TroubleShootingOnHelm"
keywords = ["Linux"]
categories = ["Linux"]
+++
这几天在试用helm，很有意思的包管理系统，让容器解决方案的落地门槛一下降了很多。然后我在搭建前端可视化的仓库解决方案，用到了monocular,
百思不得其解的是，在minikube上可以顺利部署成功的monocular,
在自己搭建的基于Ubuntu搭建的k8s集群上就是不行。    

解决方法： 用`kubectl get pods`来看，总是mongodb部署不成功。    

用kubernetes dashboard查看pod失败的原因在与persistence volume mount不成功。    

在minikube上用`kubectl get pv`和`kubectl get
pvc`是可以看到完整的结果的，而且可以看到它使用的是hostpath的格式。    

下载monocular的charts到本地，查看目录结构:    

```
# helm fetch monocular/monocular
# tree
.
├── charts
│   └── mongodb
│       ├── Chart.yaml
│       ├── README.md
│       ├── templates
│       │   ├── deployment.yaml
│       │   ├── _helpers.tpl
│       │   ├── NOTES.txt
│       │   ├── pvc.yaml
│       │   ├── secrets.yaml
│       │   └── svc.yaml
│       └── values.yaml
├── Chart.yaml
├── README.md
├── requirements.lock

```
而后我们查看charts里关于持久化存储的声明，发现是在charts/mongodb下所设置的，    

```
# cat values.yaml  | grep -i persistence -A5
    ## Enable persistence using Persistent Volume Claims
    ## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
    ##
    persistence:
      enabled: true
      ## mongodb data Persistent Volume Storage Cla
```
于是我们用以下的命令来重新安装此包:    

```
# helm ls
.... // panda stands for the installed helm instance
# helm delete panda
# helm install --name=monkey --set "persistence.enabled=false,mongodb.persistence.enabled=false"  monocular/monocular
```
现在刷新系统，发现已经安装成功了:    

```
# kubectl get pods
NAME                                                        READY     STATUS    RESTARTS   AGE
monkey-mongodb-66fd888d4-k66tg                              1/1       Running   0          19m
monkey-monocular-api-5fd987957-rtmqq                        1/1       Running   6          19m
monkey-monocular-api-5fd987957-wqxds                        1/1       Running   6          19m
monkey-monocular-prerender-6b7cb5cc98-gxs8b                 1/1       Running   0          19m
monkey-monocular-ui-8c776fd89-5hbcg                         1/1       Running   0          19m
monkey-monocular-ui-8c776fd89-gz8jm                         1/1       Running   0          19m
my-release-nginx-ingress-controller-74c748b9fb-9xtfv        1/1       Running   7          17h
my-release-nginx-ingress-default-backend-64f764b667-gxkht   1/1       Running   4          17h
# kubectl get ingress
NAME               HOSTS     ADDRESS         PORTS     AGE
monkey-monocular   *         10.15.205.200   80        20m
```
打开网页，发现可以访问到monocular, 然而其charts列表暂时无法显示， why?



迅速部署应用，避免每次重新拉取镜像：    

```
# helm install --name=tiger --set "persistence.enabled=false,mongodb.persistence.enabled=false,pullPolicy=IfNotPresent,api.image.pullPolicy=IfNotPresent,ui.image.pullPolicy=IfNotPresent,prerender.image.pullPolicy=IfNotPresent" monocular/monocular
```
