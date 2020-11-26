+++
title= "WorkingTipsOnAiK8s"
date = "2020-11-25T11:25:21+08:00"
description = "WorkingTipsOnAiK8s"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Images
Pull:    

```
# sudo docker pull gcr.io/google-containers/ubuntu-nvidia-driver-installer@sha256:7df76a0f0a17294e86f691c81de6bbb7c04a1b4b3d4ea4e7e2cccdc42e1f6d63
# sudo docker pull k8s.gcr.io/nvidia-gpu-device-plugin@sha256:0842734032018be107fa2490c98156992911e3e1f2a21e059ff0105b07dd8e9e
# sudo docker pull nvidia/dcgm-exporter:1.4.3
```

docker tag and push to inner repository:    

```
root@focal-1:/home/test# docker tag a71c5a2117c0 gcr.io/google-containers/ubuntu-nvidia-driver-installer:rong
root@focal-1:/home/test# docker tag 6527686cc4d1 k8s.gcr.io/nvidia-gpu-device-plugin:rong
root@focal-1:/home/test# docker push gcr.io/google-containers/ubuntu-nvidia-driver-installer:rong
The push refers to repository [gcr.io/google-containers/ubuntu-nvidia-driver-installer]
d09b6f591248: Pushed 
72f630252ba2: Pushed 
68dda0c9a8cd: Pushed 
f67191ae09b8: Pushed 
b2fd8b4c3da7: Pushed 
0de2edf7bff4: Pushed 
rong: digest: sha256:7df76a0f0a17294e86f691c81de6bbb7c04a1b4b3d4ea4e7e2cccdc42e1f6d63 size: 1570
root@focal-1:/home/test# docker push k8s.gcr.io/nvidia-gpu-device-plugin:rong
The push refers to repository [k8s.gcr.io/nvidia-gpu-device-plugin]
179f02762b1a: Pushed 
cd7100a72410: Pushed 
rong: digest: sha256:0842734032018be107fa2490c98156992911e3e1f2a21e059ff0105b07dd8e9e size: 739
```

### Tips
因为该插件比较老，因而不再需要使用，正确的方式参考：   

https://github.com/NVIDIA/k8s-device-plugin
