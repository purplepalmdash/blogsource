+++
title= "KubeQuickRun"
date = "2021-02-03T14:38:00+08:00"
description = "KubeQuickRun"
keywords = ["Technology"]
categories = ["Technology"]
+++
Via following tips we could quickly run apps in k8s:    

```
# kubectl create deployment xxxx --image=xxxx:16.04 -- sleep 3000
# kubectl get deployment
# kubectl get deployment xxxx -oyaml --export>kkk.yaml
# vim kkk.yaml
command: ["sleep"]
args: ["3600000"]
# kubectl delete -f kkk.yaml
# kubectl create -f kkk.yaml
# kubectl scale deployment xxxx -replicas=4
```
