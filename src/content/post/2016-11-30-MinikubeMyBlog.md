+++
categories = ["Virtualization"]
date = "2016-11-30T10:44:10+08:00"
description = "Make blog running in minikube"
keywords = ["Virtualization"]
title = "MinikubeMyBlog"

+++
### Generate blog
Generate the static blog via:    

```
# hugo --theme=hyde-a
```
### Persist Volume
Define a pv:    

```
$ vim blog.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pvblog
  labels:
    type: local
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/hugoblog"
```
Create this pv:    

```
$ kubectl create -f blog.yaml
persistentvolume "pvblog" created
```

Create a pv claim:    

```
$ vim blogclaim.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: blogclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
Create this pv claim:    

```
$ kubectl create -f ./blogclaim.yaml
persistentvolumeclaim "blogclaim" created
```
Examine the result:    

```
$ kubectl get pv
NAME      CAPACITY   ACCESSMODES   STATUS    CLAIM               REASON    AGE
pvblog    5Gi        RWO           Bound     default/blogclaim             4m
$ kubectl get pvc
NAME        STATUS    VOLUME    CAPACITY   ACCESSMODES   AGE
blogclaim   Bound     pvblog    5Gi        RWO           2m
```

Upload your blog website into `/data/hugoblog`.   

Create a pod definition:    

```
$ vim hugo.yaml
kind: Pod
apiVersion: v1
metadata:
  name: hugoblog
  labels:
    name: hugoblog
spec:
  containers:
    - name: hugocontainer
      image: nginx
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: pvblog
  volumes:
    - name: pvblog
      persistentVolumeClaim:
       claimName: blogclaim
```
Creat this pod via:    

```
$ kubectl create -f hugo.yaml
pod "hugoblog" created
$ kubectl get pod
NAME       READY     STATUS    RESTARTS   AGE
hugoblog   1/1       Running   0          <invalid>
```

Expose service:    

```
$ vim nginx.json
{
  "kind": "Service",
  "apiVersion": "v1",
  "metadata": {
    "name": "frontendservice"
  },
  "spec": {
    "ports": [
      {
        "protocol": "TCP",
        "port": 3000,
        "targetPort": "http-server"
      }
    ],
    "type": "LoadBalancer",
    "selector": {
      "name": "hugoblog"
    }
  }
}
```
Creat the service via this json file:    

```
$ kubectl create -f nginx.json
```
Get the service status, and access it via minikube command:    

```
$  kubectl get service
NAME              CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
frontendservice   10.0.0.217   <pending>     3000/TCP   2m
kubernetes        10.0.0.1     <none>        443/TCP    1d
$ minikube service frontendservice --url
http://192.168.99.101:31521
```
Open your browser and navigate to the corresponding url then you could get the
website running.    

### port-forward
Use following command, forward the local flows to pod:     

```
$ kubectl port-forward hugoblog 8078:80
``` 
Now open your browser visit `http://localhost:8078`, then you could visit the
blog. 
