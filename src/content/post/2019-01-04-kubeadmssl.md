+++
title = "kubeadmssllifetime"
date = "2019-01-04T12:18:49+08:00"
description = "kubeadmssllifetime"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Reason
The ssl lifetime is only 1 year, we need to changes it to 100 years.    

### Steps
Check out the specific version:    

```
# git clone  https://github.com/kubernetes/kubernetes
# git checkout tags/v1.12.3 -b 1.12.3_local
```
Now edit the cert.go file:    

```
# vim vendor/k8s.io/client-go/util/cert/cert.go

		NotAfter:              now.Add(duration365d * 100).UTC(),    // line 66
		NotAfter:     time.Now().Add(duration365d * 100).UTC(),  // line 111
	maxAge := time.Hour * 24 * 365 * 100         // one year self-signed certs  // line 96
		maxAge = 100 * time.Hour * 24 * 365 // 100 years fixtures  // line 110
		NotAfter:    validFrom.Add(100 * maxAge), // line 152, 124
```
Then build using following command:    

```
# make all WHAT=cmd/kubeadm GOFLAGS=-v
# ls  _output/bin/kubeadm
```
Now using the newly generated kubeadm for replacing kubespray's kubeadm.    

Also you have to change the sha256sum of the kubeadm which exists in
`roles/download/defaults/main.yml`:    


```
kubeadm_checksums:
  v1.12.4: bc7988ee60b91ffc5921942338ce1d103cd2f006c7297dd53919f4f6d16079fa
  #v1.12.4: 674ad5892ff2403f492c9042c3cea3fa0bfa3acf95bc7d1777c3645f0ddf64d7
```
deploy a cluster again, this time you will get a 100-year signature:    

```
root@k8s-1:/etc/kubernetes/ssl# pwd
/etc/kubernetes/ssl
root@k8s-1:/etc/kubernetes/ssl# for i in `ls *.crt`; do openssl x509 -in $i -noout -dates; done | grep notAfter
notAfter=Dec 11 05:34:10 2118 GMT
notAfter=Dec 11 05:34:11 2118 GMT
notAfter=Dec 11 05:34:10 2118 GMT
notAfter=Dec 11 05:34:11 2118 GMT
notAfter=Dec 11 05:34:12 2118 GMT
```

### v1.12.5
Update the v1.12.5   

```
#  git remote -v
#  git fetch origin
#  git checkout tags/v1.12.5 -b 1.12.5_local
# git branch
  1.12.3_local
  1.12.4_local
* 1.12.5_local
  master
......make some changes.....
# make all WHAT=cmd/kubeadm GOFLAGS=-v
# ls  _output/bin/kubeadm

```

### 4. kubeadm git tree state
Modify the file `hack/lib/version.sh`:    

```
  if [[ -n ${KUBE_GIT_COMMIT-} ]] || KUBE_GIT_COMMIT=$("${git[@]}" rev-parse "HEAD^{commit}" 2>/dev/null); then
    if [[ -z ${KUBE_GIT_TREE_STATE-} ]]; then
      # Check if the tree is dirty.  default to dirty
      if git_status=$("${git[@]}" status --porcelain 2>/dev/null) && [[ -z ${git_status} ]]; then
        KUBE_GIT_TREE_STATE="clean"
      else
        KUBE_GIT_TREE_STATE="clean"
      fi
    fi
```

### golang issue
build kubeadm 1.14.1 requires golang newer than golang 1.12.    

```
# wget https://dl.google.com/go/go1.12.2.linux-amd64.tar.gz
# tar -xvf go1.12.2.linux-amd64.tar.gz
# sudo mv go /usr/local
# vim ~/.bashrc
export GOROOT=/usr/local/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
export GOPATH=/root/go/
# source ~/.bashrc
# go version
go version go1.12.2 linux/amd64
```

Now you could use the newer golang builder for building the v1.14.1 kubeadm.  


### 1.14.1 kubeadm timestamp
Before:    

```
# pwd
/etc/kubernetes/ssl
# for i in `ls *.crt`; do openssl x509 -in $i -noout -dates; done | grep notAfter
notAfter=May  4 07:20:04 2020 GMT
notAfter=May  4 07:20:03 2020 GMT
notAfter=May  2 07:20:03 2029 GMT
notAfter=May  2 07:20:04 2029 GMT
notAfter=May  4 07:20:05 2020 GMT
```
After replacement:    

```
notAfter=May  4 08:13:02 2020 GMT
notAfter=May  4 08:13:02 2020 GMT
notAfter=Apr 11 08:13:01 2119 GMT
notAfter=Apr 11 08:13:02 2119 GMT
notAfter=May  4 08:13:03 2020 GMT
```

Seems failed, so I have to change again.   

Add modification:    

```
./cmd/kubeadm/app/util/pkiutil/pki_helpers.go
                NotAfter:     time.Now().Add(duration365d * 100 ).UTC(),  // line 578

```

### arm64(kubernetes 1.14.3 version)
golang 1.12.2 arm64 version download:     

```
# wget https://dl.google.com/go/go1.12.2.linux-arm64.tar.gz
# tar xzvf go1.12.2.linux-arm64.tar.gz
# sudo mv go /usr/local
# vim ~/.bashrc
export GOROOT=/usr/local/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
export GOPATH=/root/go/
# source ~/.bashrc
# go version
go version go1.12.2 linux/arm64
```
Download the k8s 1.14.3 source code and unzip it:     

```
# unzip kubernetes-1.14.3.zip
# cd kubernetes-1.14.3
```
modify the `hack/lib/version.sh` `KUBE_GIT_TREE_STATE` all to `clean`.     

Also change following two files:    

```
root@arm02:~/Code/kubernetes-1.14.3# vim cmd/kubeadm/app/util/pkiutil/pki_helpers.go
root@arm02:~/Code/kubernetes-1.14.3# vim vendor/k8s.io/client-go/util/cert/cert.go
root@arm02:~/Code/kubernetes-1.14.3#  make all WHAT=cmd/kubeadm GOFLAGS=-v
```

![/images/2019_07_03_15_25_16_903x178.jpg](/images/2019_07_03_15_25_16_903x178.jpg)

### v1.15.3
Via following steps:    

```
# cd YOURKUBERNETES_FOLDER
# git fetch origin
# git checkout tags/v1.15.3 -b 1.15.3_local
# vim hack/lib/version.sh
      if git_status=$("${git[@]}" status --porcelain 2>/dev/null) && [[ -z ${git_status} ]]; then
        KUBE_GIT_TREE_STATE="clean"
      else
        KUBE_GIT_TREE_STATE="clean"
# vim cmd/kubeadm/app/constants/constants.go
        CertificateValidity = time.Hour * 24 * 365 *100
# vim vendor/k8s.io/client-go/util/cert/cert.go
edit the same as in v1.12.5
		NotAfter:              now.Add(duration365d * 100).UTC(),    // line 66
		NotAfter:     time.Now().Add(duration365d * 100).UTC(),  // line 111
	maxAge := time.Hour * 24 * 365 * 100         // one year self-signed certs  // line 96
		maxAge = 100 * time.Hour * 24 * 365 // 100 years fixtures  // line 110
		NotAfter:    validFrom.Add(100 * maxAge), // line 152, 124
# make all WHAT=cmd/kubeadm GOFLAGS=-v
# ls  _output/bin/kubeadm
```

注意: v1.15.3中, CertificateValidity变量定义为100年后，不需修改`pki_helper.go`文件内容。    
