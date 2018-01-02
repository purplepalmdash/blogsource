+++
categories = ["Virtualization"]
date = "2016-12-27T19:06:23+08:00"
description = "Virsualizer for Kubernetes"
keywords = ["Virtualization"]
title = "VisualizerForKubernetes"

+++
Download the source code from github:    

```
$ git clone https://github.com/saturnism/gcp-live-k8s-visualizer
$ kubectl proxy ---www=./
```
Create the service/pods like the ones in examples, then you get the beautiful
view for your pods and services:    

![/images/2016_12_27_18_06_46_1061x521.jpg](/images/2016_12_27_18_06_46_1061x521.jpg)    

Change the rcs to 10:    

```
$ kubectl scale rc nginx --replicas=10
```
Now the image changes to:    

![/images/2016_12_27_18_06_57_1852x550.jpg](/images/2016_12_27_18_06_57_1852x550.jpg)
