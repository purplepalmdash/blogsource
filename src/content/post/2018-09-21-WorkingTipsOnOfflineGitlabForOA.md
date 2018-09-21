+++
title = "OAGitlabK8s"
date = "2018-09-21T09:54:20+08:00"
description = "OAGitlabK8s"
keywords = ["Linux"]
categories = ["Linux"]
+++
### gitlab/gitlab-runner
Install via:    

```
$ sudo yum install -y curl policycoreutils-python openssh-server
$  sudo systemctl enable sshd
$  sudo systemctl start sshd
$  curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
$  sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ce
$  curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
$  sudo yum install -y gitlab-runner
```
Edit gitlab.rb file:   

```
# vim /etc/gitlab.rb
external_url 'http://192.168.122.160'

prometheus['enable'] = true
prometheus['monitor_kubernetes'] = true
prometheus['listen_address'] = "0.0.0.0:9090"
prometheus['username'] = 'gitlab-prometheus'
prometheus['uid'] = nil
prometheus['gid'] = nil
prometheus['shell'] = '/bin/sh'
prometheus['home'] = '/var/opt/gitlab/prometheus'
prometheus['log_directory'] = '/var/log/gitlab/prometheus'
prometheus['scrape_interval'] = 15
prometheus['scrape_timeout'] = 15
prometheus['chunk_encoding_version'] = 2
# To completely disable prometheus, and all of it's exporters, set to false
prometheus_monitoring['enable'] = true
gitlab_workhorse['auth_backend']= "http://localhost:8081"
unicorn['listen'] = 'localhost'
unicorn['port'] = 8081
gitlab_rails['gitlab_shell_ssh_port'] = 2222
#  gitlab-ctl reconfigure && gitlab-ctl restart
```
Now setup the password and re-login then you will see the gitlab running.    

![/images/2018_09_21_10_51_14_407x294.jpg](/images/2018_09_21_10_51_14_407x294.jpg)

### Project Setup
Add project `testci`:    

![/images/2018_09_21_10_53_57_476x253.jpg](/images/2018_09_21_10_53_57_476x253.jpg)
Add the ssh key:    

![/images/2018_09_21_10_53_34_605x494.jpg](/images/2018_09_21_10_53_34_605x494.jpg)
Using the existing project:    

```
$ cd presentation-gitlab-k8s
$ git init
$ git add .
$ git commit -m "Initial"
$ git remote add origin ssh://git@192.192.185.92:222/root/testci.git
$ git push origin master
```
Your project is up-to-date but the ci/cd is not triggered.    

![/images/2018_09_21_11_02_02_898x181.jpg](/images/2018_09_21_11_02_02_898x181.jpg)

### Integration Of K8s
Options->Kubernetes:    

![/images/2018_09_21_11_02_43_208x179.jpg](/images/2018_09_21_11_02_43_208x179.jpg)

Add Kubernetes cluster:    

![/images/2018_09_21_11_03_01_487x211.jpg](/images/2018_09_21_11_03_01_487x211.jpg)

Input APIURL/CA/Token/Namespce, etc:    

![/images/2018_09_21_11_09_14_565x550.jpg](/images/2018_09_21_11_09_14_565x550.jpg)

CA/Token could referes to:    

[https://edenmal.moe/post/2018/GitLab-Kubernetes-Using-GitLab-CIs-Kubernetes-Cluster-feature/#step-3-get-the-kubernetes-ca-certificate](https://edenmal.moe/post/2018/GitLab-Kubernetes-Using-GitLab-CIs-Kubernetes-Cluster-feature/#step-3-get-the-kubernetes-ca-certificate)    

### Offlien helm/charts
Create new registry namespace in existing environment:    

![/images/2018_09_21_11_16_56_508x310.jpg](/images/2018_09_21_11_16_56_508x310.jpg)

This repository namespace is for holding all of the docker images.   


### Steps
#### helm tiller
Before install, create service accounta:    

```
 kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts
```
Get the helm version and adjust the helm definition:    

```
# cat /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm.rb | grep -i helm_version
HELM_VERSION = '2.7.2'.freeze
# wget https://kubernetes-helm.storage.googleapis.com/helm-v2.7.2-linux-amd64.tar.gz
# scp ./helm-v2.7.2-linux-amd64.tar.gz to your static web server(http://portus.ooooooo.com:8988)
# vim  /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/base_command.rb 
-            ALPINE_VERSION=$(cat /etc/alpine-release | cut -d '.' -f 1,2)
-            echo http://mirror.clarkson.edu/alpine/v$ALPINE_VERSION/main >> /etc/apk/repositories
-            echo http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v$ALPINE_VERSION/main >> /etc/apk/repositories
-            apk add -U wget ca-certificates openssl >/dev/null
-            wget -q -O - https://kubernetes-helm.storage.googleapis.com/helm-v#{Gitlab::Kubernetes::Helm::HELM_VERSION}-linux-amd64.tar.gz | tar zxC /tmp >/dev/null
+            wget -q -O - http://portus.ooooooo.com:8888/helm-v2.7.2-linux-amd64.tar.gz | tar zxC /tmp >/dev/null
# vim /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/install_command.rb
-          'helm init --client-only >/dev/null'
+          'helm init --stable-repo-url http://portus.ooooooo.com:8988 --tiller-image portus.ooooooo.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.2 >/dev/null'
# /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/init_command.rb
-          "helm init >/dev/null"
+          'helm init --stable-repo-url http://portus.ooooooo.com:8988 --tiller-image portus.ooooooo.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.2 >/dev/null'
```
Upload necessary docker images:    

```
# docker push portus.ooooooo.com:5000/kubesprayns/alpinewithwget:3.6
# docker push portus.ooooooo.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.2
```
#### Uploading Changed charts

```
# curl --data-binary "@gitlab-runner-0.1.33.tgz" http://portus.ooooooo.com:8988/api/charts
# curl --data-binary "@nginx-ingress-0.28.2.tgz" http://portus.ooooooo.com:8988/api/charts
# curl --data-binary "@prometheus-7.1.0.tgz" http://portus.ooooooo.com:8988/api/charts
```

#### gitlab runner for k8s

Change the charts repository:    

```
$ vim /opt/gitlab/embedded/service/gitlab-rails/app/models/clusters/applications/runner.rb
-        'https://charts.gitlab.io'
+        'http://portus.ooooooo.com:8988'
```
Upload docker images:    

```

```
