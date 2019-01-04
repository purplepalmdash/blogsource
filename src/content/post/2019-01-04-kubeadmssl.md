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

		NotAfter:              now.Add(duration365d * 100).UTC(),
		NotAfter:     time.Now().Add(duration365d * 100).UTC(),
	maxAge := time.Hour * 24 * 365 * 100         // one year self-signed certs
		maxAge = 100 * time.Hour * 24 * 365 // 100 years fixtures
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
