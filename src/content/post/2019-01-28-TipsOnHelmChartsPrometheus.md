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

### Issue

```
quay.io/coreos/configmap-reload:v0.0.1

```
