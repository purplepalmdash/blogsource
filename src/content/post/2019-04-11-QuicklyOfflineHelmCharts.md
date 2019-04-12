+++
title = "QuicklyOfflineHelmCharts"
date = "2019-04-11T22:50:33+08:00"
description = "QuicklyOfflineHelmCharts"
keywords = ["Linux"]
categories = ["Linux"]
+++
Following is the tips:    

Create the minikube environment, and enable the helm/charts:    

```
#  minikube start --cpus 4 --memory 8192 --disk-size 60g
# helm init
```
Download the prometheus-operator from the official github repository, use
`dependency update` for updating the dependency locally:    

```
# cd prometheus-operator
# helm dependency update
```
Record the docker images before deployment:    

```
# eval $(minikube docker-env)
# docker images>before.txt
```
Now deploy the helm/charts using:    

```
#  helm install --name newprom .
```

When all of the items were deployed, record the images via:    

```
# docker images>after.txt
```

With the helm/charts folder and the `before.txt` and `after.txt` you could
making this helm/charts working offline.   

```
# docker save -o prometheus-operator.tar grafana/grafana:6.0.2 kiwigrid/k8s-sidecar:0.0.13 quay.io/coreos/prometheus-config-reloader:v0.29.0 quay.io/coreos/prometheus-operator:v0.29.0 quay.io/prometheus/alertmanager:v0.16.1 quay.io/prometheus/prometheus:v2.7.1 k8s.gcr.io/kube-state-metrics:v1.5.0 quay.io/prometheus/node-exporter:v0.17.0 quay.io/coreos/configmap-reload:v0.0.1
# xz prometheus-operator.tar
# tag and push
```
