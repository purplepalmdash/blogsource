+++
title = "MakeChartsRepositoryLocal"
date = "2018-01-19T09:01:25+08:00"
description = "MakeChartsRepositoryLocal"
keywords = ["Linux"]
categories = ["Technology"]
+++
在离线环境下，如何将在线的charts仓库的内容本地化？    

举google的charts里的gitlab为例说明, 原网址在:    

[https://github.com/kubernetes/charts](https://github.com/kubernetes/charts)   

gitlab-ce的项在:    

[https://github.com/kubernetes/charts/tree/master/stable/gitlab-ce](https://github.com/kubernetes/charts/tree/master/stable/gitlab-ce)   

下载仓库的定义文件:    

```
# wget https://kubernetes-charts.storage.googleapis.com/index.yaml
```

`index.yaml`文件大小为900多K，里面包含了google提供的k8s-charts仓库里的所有条目，既然我们需要的是gitlab-ce条目，那我们删除掉其他条目:    

```
apiVersion: v1
entries:
  gitlab-ce:
  - created: 2017-11-14T00:04:13.262486613Z
......

    version: 0.1.0
generated: 2018-01-18T18:49:08.04868721Z
```
查看该文件，发现我们需要的tgz文件及icon文件，手动下载之：   

```
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.2.1.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.2.0.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.12.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.11.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.10.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.9.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.8.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.7.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.6.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.5.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.4.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.3.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.2.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.1.tgz
# wget https://kubernetes-charts.storage.googleapis.com/gitlab-ce-0.1.0.tgz
# wget https://gitlab.com/uploads/group/avatar/6543/gitlab-logo-square.png
```
而后手动解压缩开所有tgz文件，下载对应的镜像，并下载为离线文件，安装到工作节点上.    

TODO: 这里应该有改造，需要有自己的本地仓库，推送到本地仓库，并对应更改镜像名为本地仓库。    

接着，替换出index.yaml为本地的位于web server上的tgz文件和icon文件即可。
