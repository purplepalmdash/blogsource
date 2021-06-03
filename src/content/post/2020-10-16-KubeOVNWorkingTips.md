+++
title= "KubeOVNWorkingTips"
date = "2020-10-16T08:27:51+08:00"
description = "KubeOVNWorkingTips"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Server side(k8s)
`kkk.yaml` defined the subnet created via kubeovn
```
apiVersion: kubeovn.io/v1
kind: Subnet
metadata:
  name: etc
spec:
  protocol: IPv4
  default: false
  namespaces:
  - etl
  - etl1
  cidrBlock: 100.64.0.0/16
  gateway: 100.64.0.1
  excludeIps:
  - 100.64.0.1
  private: false
  gatewayType: distributed
  natOutgoing: false
```
Create the subnet via `kubectl create -f kkk.yaml`, then you could view the subnet via:    

```
# kubectl get subnet
NAME	PROTOCOL	CIDR		PRIVATE	NAT	DEFAULT	GATEWAYTYPE	USED AVAILABLE
etc	IPV4		100.64.0.0/16	false	false	false	distributed	1	65532
```
Create namespace via `kubectl create ns etl` and `kubectl create ns etl1`, then run a deployment in these 2 namespace:    

```
# kubectl run nginxetl --image=nginx:1.17 --namespace etl
# kubectl get pod -n etl -o wide
The pod's ip address is 100.64.0.3
```
### Client Side(outer space machines)
Add route via:    

```
# route add -net 100.64.0.0/16 gw 192.192.xxx.xxx
# curl 100.64.0.3
```
