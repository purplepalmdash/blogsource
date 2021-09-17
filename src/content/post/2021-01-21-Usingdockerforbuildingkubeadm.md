+++
title= "Usingdockerforbuildingkubeadm"
date = "2021-01-21T15:04:12+08:00"
description = "Usingdockerforbuildingkubeadm"
keywords = ["Technology"]
categories = ["Technology"]
+++
X86 version kubeadm building process:    

```
# docker run -it ubuntu:18.04 /bin/bash
# cat /etc/issue
Ubuntu 18.04.5 LTS \n \l
# apt-get update -y
# apt-get install -y wget unzip vim build-essential rsync
# wget https://github.com/kubernetes/kubernetes/archive/v1.19.7.zip
# wget https://golang.org/dl/go1.15.7.linux-amd64.tar.gz
# tar -C /usr/local -xzf go1.15.7.linux-amd64.tar.gz
# export PATH=$PATH:/usr/local/go/bin
# go version
go version go1.15.7 linux/amd64
# cd kubernetes-v1.19.7
# vim cmd/kubeadm/app/constants/constants.go
CertificateValidity = time.Hour * 24 * 365 * 100
#  vim vendor/k8s.io/client-go/util/cert/cert.go
func NewSelfSignedCACert
NotAfter: 	now.Add(duration365d * 100).UTC(),
func GenerateSelfSignedCertKeyWithFixtures
maxAge := 100 * time.Hour * 24 * 365
# make all WHAT=cmd/kubeadm
# _output/bin/kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.7", GitCommit:"1dd5338295409edcfff11505e7bb246f0d325d15", GitTreeState:"archive", BuildDate:"2021-01-21T07:15:36Z", GoVersion:"go1.15.7", Compiler:"gc", Platform:"linux/amd64"}
# cp _output/bin/kubeadm ..
```

Arm64 version kubeadm building process:    

Edit files and make changes:    

```
# vim hack/make-rules/cross.sh
    make all WHAT="${KUBE_SERVER_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_SERVER_PLATFORMS[*]}"
    
    #make all WHAT="${KUBE_NODE_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_NODE_PLATFORMS[*]}"
    #
    #make all WHAT="${KUBE_CLIENT_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_CLIENT_PLATFORMS[*]}"
    #
    #make all WHAT="${KUBE_TEST_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_TEST_PLATFORMS[*]}"
    #
    #make all WHAT="${KUBE_TEST_SERVER_TARGETS[*]}" KUBE_BUILD_PLATFORMS="${KUBE_TEST_SERVER_PLATFORMS[*]}"
#  vim hack/lib/golang.sh
    readonly KUBE_SUPPORTED_SERVER_PLATFORMS=(
    #  linux/amd64
    #  linux/arm
      linux/arm64
    #  linux/s390x
    #  linux/ppc64le
    )
    
    //.............
    
    kube::golang::server_targets() {
      local targets=(                       
      #  cmd/kube-proxy                                                        
      #  cmd/kube-apiserver                 
      #  cmd/kube-controller-manager                                 
      #  cmd/kubelet 
        cmd/kubeadm
      #  cmd/kube-scheduler
      #  vendor/k8s.io/apiextensions-apiserver
      #  cluster/gce/gci/mounter
      )
```
Build:    

```
# make cross
```

### v1.20.7 update
Have to change go version to v1.17 for building, also notice the memory usage. 

```
apt-get update -y && apt-get install -y wget unzip vim build-essential rsync
wget https://github.com/kubernetes/kubernetes/archive/v1.20.7.zip
wget https://golang.org/dl/go1.17.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.17.linux-amd64.tar.gz 
export PATH=$PATH:/usr/local/go/bin
go version
unzip v1.20.7.zip 
cd kubernetes-1.20.7/
vim cmd/kubeadm/app/constants/constants.go
vim vendor/k8s.io/client-go/util/cert/cert.go
make all WHAT=cmd/kubeadm
cp _output/bin/kubeadm ..

```
