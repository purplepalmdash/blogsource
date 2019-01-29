+++
title = "TipsOnHelmChartsPrometheus"
date = "2019-01-28T15:06:56+08:00"
description = "TipsOnHelmChartsPrometheus"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Upgrade helm
Download the helm from github, and extracted to your system PATH, then:    

```
# helm init --upgrade
```
Notice in your k8s cluster your tiller will be upgraded:    

```
kube-system   tiller-deploy-68b77f4c57-dwd47                                  0/1       ContainerCreating   0          13s
```

Examine the upgraded version:    

```
# helm version
Client: &version.Version{SemVer:"v2.12.2", GitCommit:"7d2b0c73d734f6586ed222a567c5d103fed435be", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.12.2", GitCommit:"7d2b0c73d734f6586ed222a567c5d103fed435be", GitTreeState:"clean"}
```

### Working Issue
Using stable/prometheus-operator.   

Fetch the helm/charts package via:    

```
# helm fetch stable/prometheus-operator
# ls
prometheus-operator-1.9.0.tgz
```
Deploy it on the minikube and you will get all of the images, then save it
via:    

```
eval $(minikube docker-env)
docker save -o prometheus-operator.tar grafana/grafana:5.4.3 quay.io/prometheus/node-exporter:v0.17.0 quay.io/coreos/prometheus-config-reloader:v0.26.0 quay.io/coreos/prometheus-operator:v0.26.0 quay.io/prometheus/alertmanager:v0.15.3 quay.io/prometheus/prometheus:v2.5.0 kiwigrid/k8s-sidecar:0.0.6 quay.io/coreos/kube-state-metrics:v1.4.0 quay.io/coreos/configmap-reload:v0.0.1
xz prometheus-operator.tar
```
Then we could write the offline scripts for deploying this helm/charts.    
