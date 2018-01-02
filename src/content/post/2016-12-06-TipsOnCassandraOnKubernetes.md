+++
categories = ["Virtualization"]
date = "2016-12-06T19:27:19+08:00"
description = "Run cassandra on kubernetes"
keywords = ["Virtualization"]
title = "TipsOnCassandraOnKubernetes"

+++
First download the image from gcr.io:    

```
$ sudo docker pull gcr.io/google-samples/cassandra:v11
```
Create a replicas using following yaml file:    

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: cassandra
  # The labels will be applied automatically
  # from the labels in the pod template, if not set
  # labels:
    # app: cassandra
spec:
  replicas: 1
  # The selector will be applied automatically
  # from the labels in the pod template, if not set.
  # selector:
      # app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
        - command:
            - /run.sh
          resources:
            limits:
              cpu: 0.5
          env:
            - name: MAX_HEAP_SIZE
              value: 512M
            - name: HEAP_NEWSIZE
              value: 100M
            - name: CASSANDRA_SEED_PROVIDER
              value: "io.k8s.cassandra.KubernetesSeedProvider"
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          image: gcr.io/google-samples/cassandra:v11
          name: cassandra
          ports:
            - containerPort: 7000
              name: intra-node
            - containerPort: 7001
              name: tls-intra-node
            - containerPort: 7199
              name: jmx
            - containerPort: 9042
              name: cql
          volumeMounts:
            - mountPath: /cassandra_data
              name: data
      volumes:
        - name: data
          emptyDir: {}
```
Create rc:    

```
$ kubectl create -f cassandra-controller.yaml
```
Also create a service using following yaml definition:    

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: cassandra
  name: cassandra
spec:
  clusterIP: None
  ports:
    - port: 9042
  selector:
    app: cassandra
```
Start the service via:    

```
$ kubectl create -f cassandra-service.yaml 
```

Now scale the replica via:    

```
$  kubectl scale rc cassandra --replicas=2 
replicationcontroller "cassandra" scaled
```
Get the pods name and docker exec the command `nodetool status` to view the cassandra cluster status:    

```
 kubectl exec -ti cassandra-0px0a -- nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.5  65.61 KiB  32           100.0%            fe61ce57-2a8b-462c-8fbf-12c69461f17c  rack1
UN  172.17.0.6  84.76 KiB  32           100.0%            c9b78b8c-f207-41f8-b3f7-d962dbebe687  rack1
```
Now scale the replicas to 4 and examine the result:    

```
$ kubectl scale rc cassandra --replicas=4            
replicationcontroller "cassandra" scaled
$ kubectl exec -ti cassandra-0px0a -- nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.5  65.61 KiB  32           50.8%             fe61ce57-2a8b-462c-8fbf-12c69461f17c  rack1
UN  172.17.0.7  48.33 KiB  32           40.0%             2b9fcbed-e737-496e-b960-d3d32b3091f5  rack1
UN  172.17.0.6  84.76 KiB  32           55.4%             c9b78b8c-f207-41f8-b3f7-d962dbebe687  rack1
UN  172.17.0.8  58.09 KiB  32           53.8%             5ba29b11-7366-48d8-b3ed-ddefbd6f2e28  rack1
```
See now you could play happily with cassandra on kubernetes.    
