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

### Example
Git patch in different branch:   

```
# git clone https://github.com/kubernetes-sigs/kubespray.git
# git checkout tags/v2.14.0 -b 2140
 (2140) $ git apply ../../patch 
 (2140 !*%) $ vim roles/container-engine/docker/tasks/main.yml
 (2140 !*%) $ git checkout master
error: 您对下列文件的本地修改将被检出操作覆盖：
	roles/kubernetes-apps/helm/templates/tiller-clusterrolebinding.yml.j2
	roles/kubernetes/preinstall/tasks/0020-verify-settings.yml
	roles/remove-node/remove-etcd-node/tasks/main.yml
请在切换分支前提交或贮藏您的修改。
正在终止
(2140 !*%) $ git add .                                                                                                                        1 ↵
(2140 !+) $ git commit -m "modified in 2.14.0"
[2140 2d87573d] modified in 2.14.0
 19 files changed, 504 insertions(+), 427 deletions(-)
 delete mode 100644 contrib/packaging/rpm/kubespray.spec
 create mode 100644 inventory/sample/hosts.ini
 rewrite roles/bootstrap-os/tasks/main.yml (99%)
 create mode 100644 roles/bootstrap-os/tasks/main_kfz.yml
 copy roles/bootstrap-os/tasks/{main.yml => main_main.yml} (99%)
 rewrite roles/container-engine/docker/tasks/main.yml (99%)
 create mode 100644 roles/container-engine/docker/tasks/main_kfz.yml
 copy roles/container-engine/docker/tasks/{main.yml => main_main.yml} (92%)
dash@archnvme:/media/sda/git/pure/kubespray (2140) $ git checkout master
切换到分支 'master'
您的分支与上游分支 'origin/master' 一致。
 (master) $ ls
ansible.cfg          code-of-conduct.md  Dockerfile       index.html  logo         OWNERS_ALIASES             remove-node.yml   scale.yml          setup.py             Vagrantfile
ansible_version.yml  _config.yml         docs             inventory   Makefile     README.md                  requirements.txt  scripts            test-infra
cluster.yml          contrib             extra_playbooks  library     mitogen.yml  recover-control-plane.yml  reset.yml         SECURITY_CONTACTS  tests
CNAME                CONTRIBUTING.md     facts.yml        LICENSE     OWNERS       RELEASE.md                 roles             setup.cfg          upgrade-cluster.yml
(master) $ git apply ../../patch --exclude=roles/kubernetes-apps/helm/templates/tiller-clusterrolebinding.yml.j2 --exclude=roles/remove-node/remove-etcd-node/tasks/main.yml
(master !*%) $ git checkout 2140
error: 您对下列文件的本地修改将被检出操作覆盖：
	cluster.yml
	roles/bootstrap-os/tasks/main.yml
	roles/container-engine/docker/meta/main.yml
	roles/container-engine/docker/tasks/main.yml
	roles/container-engine/docker/tasks/pre-upgrade.yml
	roles/container-engine/docker/templates/docker-options.conf.j2
	roles/container-engine/docker/templates/docker.service.j2
	roles/kubernetes/node/tasks/kubelet.yml
	roles/kubernetes/preinstall/tasks/0020-verify-settings.yml
	roles/kubernetes/preinstall/tasks/0080-system-configurations.yml
	roles/kubernetes/preinstall/tasks/main.yml
请在切换分支前提交或贮藏您的修改。
error: 工作区中下列未跟踪的文件将会因为检出操作而被覆盖：
	inventory/sample/hosts.ini
	roles/bootstrap-os/tasks/main_kfz.yml
	roles/bootstrap-os/tasks/main_main.yml
	roles/container-engine/docker/tasks/main_kfz.yml
	roles/container-engine/docker/tasks/main_main.yml
请在切换分支前移动或删除。
正在终止
(master !*%) $ git add .                                                                                                                      1 ↵
(master !+) $ git commit -m "apply in master"
[master a5941286] apply in master
 17 files changed, 502 insertions(+), 426 deletions(-)
 delete mode 100644 contrib/packaging/rpm/kubespray.spec
 create mode 100644 inventory/sample/hosts.ini
 rewrite roles/bootstrap-os/tasks/main.yml (99%)
 create mode 100644 roles/bootstrap-os/tasks/main_kfz.yml
 copy roles/bootstrap-os/tasks/{main.yml => main_main.yml} (99%)
 rewrite roles/container-engine/docker/tasks/main.yml (99%)
 create mode 100644 roles/container-engine/docker/tasks/main_kfz.yml
 copy roles/container-engine/docker/tasks/{main.yml => main_main.yml} (92%)
 (master) $ git checkout 2140              
切换到分支 '2140'
 (2140) $ git checkout master
切换到分支 'master'
您的分支领先 'origin/master' 共 1 个提交。
  （使用 "git push" 来发布您的本地提交）
(master) $ pwd
/media/sda/git/pure/kubespray
```
