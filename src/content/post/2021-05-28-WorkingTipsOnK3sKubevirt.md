+++
title= "WorkingTipsOnK3sKubevirt"
date = "2021-05-28T09:05:11+08:00"
description = "WorkingTipsOnK3sKubevirt"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 目的
k3s+kubevirt，运行虚拟机工作负载. 落地平台为AllInOne节点。主要针对边缘侧算力平台落地场景。    

### 2. 环境
嵌套虚拟化环境用于承载k3s算力管控平台。运行操作系统为Ubuntu20.04, 40Core, 274G内存。   
更新： 嵌套虚拟化会产生诸多问题，导致qemu无法启动，因而后面我采用在物理机上直接启动k3s的方式。   

物理机环境：

```
Intel(R) Xeon(R) Gold 5118 CPU @ 2.30GHz
376G memory
2T nvme ssd
CentOS 7.6.1810
kernel: 4.19.12-1.el7.elrepo.x86_64
```

### 3. 步骤
注：以下为虚拟化环境，未安装成功（因后续有嵌套虚拟化导致qemu无法启动的问题）    
安装步骤:    

```
# apt-get update -y && apt-get upgrade -y
# export http_proxy export https_proxy
# curl -sfL https://get.k3s.io | sh -
# vim /etc/resolv.conf
nameserver 223.5.5.5
# systemctl stop systemd-resolved
# systemctl disable systemd-resolved
```

以下为正常过程:    

安装k3s:   

```
# yum update -y && yum install -y git
# curl -sfL https://get.k3s.io | sh -
```
安装KubeVirt:    

```
# export VERSION=v0.41.0
# kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
# kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```
安装`virtctl`用于控制KubeVirt虚拟机，这里使用Kubernetes的插件管理器`Krew`来安装`virtctl`:    

```
# (   set -x; cd "$(mktemp -d)" &&   curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&   tar zxvf krew.tar.gz &&   KREW=./krew-"$(uname | tr '[:upper:]' '[:lower:]')_$(uname -m | sed -e 's/x86_64/amd64/' -e 's/arm.*$/arm/')" &&   "$KREW" install krew; )

# export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
# kubectl krew install virt
```
安装`Containerized-Data-Importer(CDI)`用于管理虚拟机的磁盘，安装步骤如下:    

```
# export VERSION=$(curl -s https://github.com/kubevirt/containerized-data-importer/releases/latest | grep -o "v[0-9]\.[0-9]*\.[0-9]*")
# kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
# kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml
```
从`https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019` 下载Windows ISO 180天试用版, 而后上传该ISO:   

```
# Get the CDI upload proxy service IP:
 kubectl get svc -n cdi
    
 # Upload 
 kubectl virt image-upload --image-path </path/to/iso> \
     --pvc-name iso-win2k19 --access-mode ReadWriteOnce \
     --pvc-size 10G --uploadproxy-url <upload-proxy service:443> \
     --insecure --wait-secs=240
```
上传前需要确保`coredns`正常运行，否则会出现上传不成功的情况。   

PVC被设置成`ReadWriteOnce`, 因为默认的local-path storageclass 不支持更多的模式。因为我们只使用一个节点，所以这点没所谓，但是在大型的K3s集群里，需要注意PVC的属性配置。    

`virtio-container-disk `容器镜像需要被实现拉回，因为在安装的时候我们需要使用其中包含的驱动程序。    

```
# crictl pull kubevirt/virtio-container-disk
```
现在我们创建一个yaml文件用于定义需要创建的KubeVirt虚拟机(`win.yaml`):     

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: winhd
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 15Gi
  storageClassName: manual
---
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachine
metadata:
  name: win2k19-iso
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/domain: win2k19-iso
    spec:
      domain:
        cpu:
          cores: 4
        devices:
          disks:
          - bootOrder: 1
            cdrom:
              bus: sata
            name: cdromiso
          - disk:
              bus: sata
            name: harddrive
          - cdrom:
              bus: sata
            name: virtiocontainerdisk
        machine:
          type: q35
        resources:
          requests:
            memory: 8G
      volumes:
      - name: cdromiso
        persistentVolumeClaim:
          claimName: iso-win2k19
      - name: harddrive
        persistentVolumeClaim:
          claimName: winhd
      - containerDisk:
          image: kubevirt/virtio-container-disk
        name: virtiocontainerdisk
```
我们再创建一个PV用于承载该虚拟机的磁盘(`pv.yaml`):    

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/media/sda/win"
```
创建虚拟机:     

```
 kubectl apply -f pv.yaml
 kubectl apply -f win.yaml
 kubectl virt start win2k19-iso
 # If you're running this on a remote machine, use X-forwarding and
 # apt-get install virt-viewer
 kubectl virt vnc win2k19-iso
```
值得注意的是，`win.yaml`中我们只能选择`sata`作为主磁盘的格式，`kubectl virt vnc`在我的机器上无法使用，所以我用了`kubectl virt --proxy-only=true win2k19-iso`用于获取一个动态端口，而后`vncviewer`到该动态端口上去。    

完成安装后，进入`设备管理器`安装完未安装好的驱动程序。    

使用NodePort暴露安装好以后的虚拟机的RDP端口:     

```
apiVersion: v1
kind: Service
metadata:
  name: windows-nodeport
spec:
  externalTrafficPolicy: Cluster
  ports:
  - name: nodeport
    nodePort: 30000
    port: 27017
    protocol: TCP
    targetPort: 3389
  selector:
    kubevirt.io/domain: win2k19-iso
  type: NodePort
```
创建完该服务后，则可通过`xxx.xxx.xxx.xxx:30000`用于访问该虚拟机的RDP远程桌面端口了。  

创建完以后的虚拟机如图所示:    

![/images/2021_05_31_21_30_56_746x693.jpg](/images/2021_05_31_21_30_56_746x693.jpg)
 
如果是virtctl，则直接访问的方式是通过proxy:    

```
kubectl proxy --address=0.0.0.0 --accept-hosts='^*$' --port 8080
```

testvm:    

```
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: testvm
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/size: small
        kubevirt.io/domain: testvm
    spec:
      domain:
        devices:
          disks:
            - name: containerdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            bridge: {}
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: containerdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userDataBase64: SGkuXG4=
```
