+++
title= "WorkingTipsOnGPUOnCentOS7"
date = "2021-04-19T15:56:39+08:00"
description = "WorkingTipsOnGPUOnCentOS7"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 先决条件
各工作节点上需要保证内核为指定版本，并安装对应的`kernel-ml-devel/kernel-ml-headers/gcc`依赖包.   

```
# uname -a
Linux worker2 4.19.12-1.el7.elrepo.x86_64 #1 SMP Fri Dec 21 11:06:36 EST 2018 x86_64 x86_64 x86_64 GNU/Linux
# rpm -e --nodeps kernel-headers
# yum install -y kernel-ml-devel kernel-ml-headers gcc
```
手动安装Nvidia驱动:    

```
# ./NVIDIA-Linux-x86_64-460.32.03.run 
Verifying archive integrity... OK
Uncompressing NVIDIA Accelerated Graphics Driver for Linux-x86_64 460.32.03...........
..........................................................
..........................................................
```
忽略该报错:  

![/images/2021_04_19_16_53_12_622x250.jpg](/images/2021_04_19_16_53_12_622x250.jpg)

选择`NO`, 忽略安装32位兼容包:   

![/images/2021_04_19_16_53_55_639x172.jpg](/images/2021_04_19_16_53_55_639x172.jpg)

按`OK`结束安装:   

![/images/2021_04_19_16_54_23_623x183.jpg](/images/2021_04_19_16_54_23_623x183.jpg)

检查驱动是否安装成功:    

```
# nvidia-smi 
Mon Apr 19 04:55:13 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla V100-PCIE...  Off  | 00000000:00:08.0 Off |                  Off |
| N/A   31C    P0    36W / 250W |      0MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  Tesla V100-PCIE...  Off  | 00000000:00:0A.0 Off |                  Off |
| N/A   31C    P0    35W / 250W |      0MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

### 2. ccse改动
#### 2.1 创建新的离线库:   
引入`nvidia-docker2`相关的离线包并更新`k8s-offline-pkgs`仓库:    

```
# cd k8s-offline-pkgs/
# tar xzvf /root/nvidiadocker.tar.gz -C .
libnvidia-container-tools-1.3.3-1.x86_64.rpm
nvidia-docker2-2.5.0-1.noarch.rpm
libnvidia-container1-1.3.3-1.x86_64.rpm
nvidia-container-toolkit-1.4.2-2.x86_64.rpm
nvidia-container-runtime-3.4.2-1.x86_64.rpm
# createrepo .
```

Ccse console节点上替换离线包:    

```
[root@first x86_64]# pwd
/dcos/app/console/backend/webapps/repo/x86_64
[root@first x86_64]# mv k8s-offline-pkgs/ k8s-offline-pkgs.back
[root@first x86_64]# scp -r docker@10.168.100.1:/home/docker/k8s-offline-pkgs .
```

Ccse代码改动, 仅添加`nvidia-docker2`的安装：   

```
# vi /dcos/app/console/kubeadm-playbook/roles/util/docker/tasks/install.yml

  - name: <安装docker><install-docker> 安装 docker （ccse源）
    shell: yum install -y docker-ce nvidia-docker2 --disablerepo=\* --enablerepo=ccse-k8s,ccse-centos7-base
    when: "yum_repo == 'ccse'"
# vi /dcos/app/console/kubeadm-playbook/roles/util/docker/templates/daemon.json.j2
    { 
    {% if custom_image_repository != '' %}{{ docker_insecure_registry_mirrors | indent(2,true) }}{% endif %}
      "storage-driver": "{{ docker_storage_driver }}",
      "graph": "{{ hosts_datadir_map[inventory_hostname] }}/docker",
      "log-driver": "json-file",
      "log-opts": {
                "max-size": "1g"
            },
      "default-runtime": "nvidia",
      "runtimes": {
          "nvidia": {
              "path": "/usr/bin/nvidia-container-runtime",
              "runtimeArgs": []
          }
      }
    }

```

### 3. 验证
相关包位于`10.50.208.145`的`/home/docker`目录下的`nvidiadockerclassic.tar`:  

```
# ls /home/docker/nvidiadockerclassic.tar  -l -h
-rw-r--r-- 1 root root 187M Apr 19 17:49 /home/docker/nvidiadockerclassic.tar
```
#### 3.1 镜像准备
部署完毕后, ccse console节点上上传准备镜像:    

```
# tar xvf nvidiadockerclassic.tar 
nvidiadockerclassic/
nvidiadockerclassic/nvidia-device-plugin.yml
nvidiadockerclassic/k8sdeviceplugin.tar
# cd nvidiadockerclassic
# docker load<k8sdeviceplugin.tar
# docker tag nvcr.io/nvidia/k8s-device-plugin:v0.9.0 10.168.100.144:8021/nvcr.io/nvidia/k8s-device-plugin:v0.9.0
# docker push 10.168.100.144:8021/nvcr.io/nvidia/k8s-device-plugin:v0.9.0
```

#### 3.2 插件安装及验证
master节点上create `nvidia-device-plugin.yml`文件:    

```
# kubectl create -f nvidia-device-plugin.yml 
```

验证device-plugin安装成功:   

```
# kubectl  get po -A | grep device
kube-system   nvidia-device-plugin-daemonset-9mhq7           1/1     Running   0          19s
kube-system   nvidia-device-plugin-daemonset-m7txq           1/1     Running   0          19s
# kubectl logs nvidia-device-plugin-daemonset-9mhq7 -n kube-system
2021/04/19 09:53:23 Loading NVML
2021/04/19 09:53:23 Starting FS watcher.
2021/04/19 09:53:23 Starting OS watcher.
2021/04/19 09:53:23 Retreiving plugins.
2021/04/19 09:53:23 Starting GRPC server for 'nvidia.com/gpu'
2021/04/19 09:53:23 Starting to serve 'nvidia.com/gpu' on /var/lib/kubelet/device-plugins/nvidia-gpu.sock
2021/04/19 09:53:23 Registered device plugin for 'nvidia.com/gpu' with Kubelet
```
测试:   

```
# kubectl create -f test.yml
# kubectl  get po -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP               NODE             NOMINATED NODE   READINESS GATES
dcgmproftester   1/1     Running   0          19s   172.26.189.204   10.168.100.184   <none>           <none>
# nvidia-smi 
Mon Apr 19 05:54:55 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla V100-PCIE...  Off  | 00000000:00:09.0 Off |                  Off |
| N/A   56C    P0   218W / 250W |    493MiB / 32510MiB |     88%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A     14182      C   /usr/bin/dcgmproftester11         489MiB |
+-----------------------------------------------------------------------------+

```
