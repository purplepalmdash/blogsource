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
