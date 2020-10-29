+++
title= "WorkingTipsOnGitDiffPatch"
date = "2020-10-29T10:59:01+08:00"
description = "WorkingTipsOnGitDiffPatch"
keywords = ["Technology"]
categories = ["Technology"]
+++
### RONG代码架构现状
制作patch前确保Kubespray代码中目录中的软链接确实存在，而不是因为cp被替换成了实体文件。

v2.14.0上制作patch

```
# git clone https://github.com/kubernetes-sigs/kubespray.git
# git checkout tags/v2.14.0 -b 2140
```
此时检出的是v2.14.0的未修改的代码。    

该目录下替换成`3_k8s`下的代码，注意去掉部署时生成的中间文件。而后`commit`更改。    

![/images/2020_10_29_11_03_04_638x297.jpg](/images/2020_10_29_11_03_04_638x297.jpg)

制作patch文件:    

```
git diff a1f04e f0c9b1>patch1
```

### Apply patch
切换回master分支，或者直接在新目录下checkout一个新的工作目录:    

```
# git apply --check ../patch 
error: 打补丁失败：roles/kubernetes-apps/helm/templates/tiller-clusterrolebinding.yml.j2:3
error: roles/kubernetes-apps/helm/templates/tiller-clusterrolebinding.yml.j2：补丁未应用
error: 打补丁失败：roles/remove-node/remove-etcd-node/tasks/main.yml:21
error: roles/remove-node/remove-etcd-node/tasks/main.yml：补丁未应用
```
这是因为新分支(master)对比于`v2.14.0`在上述提及的文件中有更改，此时我们需要在apply的时候忽略掉这些更改:    

```
git apply  /root/patch --exclude=roles/kubernetes-apps/helm/templates/tiller-clusterrolebinding.yml.j2 --exclude=roles/remove-node/remove-etcd-node/tasks/main.yml
```
此时更改完毕后可以看到新版本的代码中已经添加了我们在旧分支上做的代码变更。    


对于有冲突的文件，需要手动解决冲突。  
