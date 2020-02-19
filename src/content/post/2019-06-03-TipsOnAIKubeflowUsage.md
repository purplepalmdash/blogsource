+++
title = "TipsOnAIKubeSprayUsage"
date = "2019-06-03T16:07:45+08:00"
description = "TipsOnAIKubeSprayUsage"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 0. 先决条件
Rongv1905部署就绪的集群，运行操作系统为ubuntu18.04.2.    

### 1. 部署准备
AI部署框架使用ansible撰写。目录架构如下, 配置好hosts.ini:     

```
# ls
ai.yaml  ansible.cfg  deploy.key  hosts.ini  roles
# cat hosts.ini
[all]
localnode-2 ansible_host=10.142.18.192 ansible_port=22 ansible_user='root' ansible_ssh_private_key_file=./deploy.key ip=10.142.18.192  ansible_ssh_user=root
localnode-3 ansible_host=10.142.18.193 ansible_port=22 ansible_user='root' ansible_ssh_private_key_file=./deploy.key ip=10.142.18.193  ansible_ssh_user=root
localnode-1 ansible_host=10.142.18.191 ansible_port=22 ansible_user='root' ansible_ssh_private_key_file=./deploy.key ip=10.142.18.191  ansible_ssh_user=root

[kube-deploy]
localnode-[1:1]

[kube-master]
localnode-[1:1]
```
部署命令:    

```
# ansible-playbook -i hosts.ini ai.yaml
```
### 2. 使用
#### 2.1 检查kubeflow组件
部署完毕后，通过`kubectl get pods`命令查看当前集群中运行的kubeflow相关组件:    

```
# kubectl get pods
NAME                                                     READY   STATUS    RESTARTS   AGE
ambassador-7df7dbd89b-64q5b                              2/2     Running   0          2m
ambassador-7df7dbd89b-b97w6                              2/2     Running   0          2m
ambassador-7df7dbd89b-gcdq7                              2/2     Running   0          2m
centraldashboard-5d7d79659c-dr29w                        1/1     Running   0          2m
inception-v1-d66f8848-lgz97                              1/1     Running   0          103s
pytorch-operator-7b7958b799-vmks4                        1/1     Running   0          88s
spartakus-volunteer-779867878-snvxz                      1/1     Running   0          2m
tf-hub-0                                                 1/1     Running   0          2m
tf-job-dashboard-554868c978-cknlz                        1/1     Running   0          2m
tf-job-operator-v1alpha2-7f7cf4dc98-9txms                1/1     Running   0          2m
```
通过`kubectl get svc`查看kubeflow引出的相关服务:    

```
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
ambassador                  ClusterIP   10.233.35.161   <none>        80/TCP            28m
ambassador-admin            ClusterIP   10.233.8.245    <none>        8877/TCP          28m
ambassador-katacoda         NodePort    10.233.38.151   <none>        30080:30396/TCP   3s
centraldashboard            ClusterIP   10.233.29.157   <none>        80/TCP            28m
centraldashboard-katacoda   NodePort    10.233.9.5      <none>        8082:31421/TCP    3s
k8s-dashboard               ClusterIP   10.233.3.208    <none>        443/TCP           28m
kubernetes                  ClusterIP   10.233.0.1      <none>        443/TCP           19d
tf-hub-0                    ClusterIP   None            <none>        8000/TCP          28m
tf-hub-lb                   ClusterIP   10.233.32.120   <none>        80/TCP            28m
tf-hub-lb-katacoda          NodePort    10.233.13.173   <none>        80:31039/TCP      3s
tf-job-dashboard            ClusterIP   10.233.22.155   <none>        80/TCP            28m
```
使用浏览器访问相应端口:    
centraldashboard-katacoda: 31421, ambassador-katacoda: 30396:    

![/images/2019_06_03_11_23_19_365x201.jpg](/images/2019_06_03_11_23_19_365x201.jpg)

tf-hub-lb-katacoda:    

![/images/2019_06_03_11_24_02_640x480.jpg](/images/2019_06_03_11_24_02_640x480.jpg)

#### 2.2  JypyterHub
Kubeflow的关键组件之一是能够通过JupyterHub运行Jupyter笔记本电脑。 Jupyter Notebook是经典的数据科学工具，用于在浏览器中记录流程时运行内联脚本和代码片段。    
可以使用kubectl get svc找到Load Balancer的IP地址,
Jypyter的登录默认使用用户名admin和空白密码:    

![/images/2019_06_03_11_34_33_374x377.jpg](/images/2019_06_03_11_34_33_374x377.jpg)

在弹出的Spawner Options中，我们填入下列字段。    

 Kubeflow在内部使用gcr.io/kubeflow-images-public/tensorflow-1.8.0-notebook-cpu:v0.2.1
Docker Image作为默认值。访问JupyterHub后，可以单击 Start My server按钮:    

![/images/2019_06_03_11_36_38_480x303.jpg](/images/2019_06_03_11_36_38_480x303.jpg)

```
root@localnode-1:~# kubectl get pods -o wide | grep jupyter-admin
jupyter-admin                                         1/1     Running   0          39s   10.233.125.9   localnode-1   <none>           <none>
```
Spawn完毕后界面如下:    

![/images/2019_06_03_11_39_04_782x214.jpg](/images/2019_06_03_11_39_04_782x214.jpg)

现在可以通过pod访问JupyterHub。您现在可以无缝地使用环境。例如，要创建新笔记本，请选择New下拉列表，然后选择Python 3内核，如下所示。

![/images/2019_06_03_11_39_56_231x244.jpg](/images/2019_06_03_11_39_56_231x244.jpg)

现在可以创建代码片段。要开始使用TensorFlow，请将下面的代码粘贴到第一个单元格并运行它。   

```
from __future__ import print_function

import tensorflow as tf

hello = tf.constant('Hello TensorFlow!')
s = tf.Session()
print(s.run(hello))
```
运行结果如下：    

![/images/2019_06_03_11_41_53_785x321.jpg](/images/2019_06_03_11_41_53_785x321.jpg)

#### 2.3 部署TensorFlow Job(TFJob)
TfJob提供了一个Kubeflow自定义资源，可以在Kubernetes上轻松运行分布式或非分布式TensorFlow作业。 TFJob控制器为master，parameter servers和worker采用YAML规范来帮助运行分布式计算。

自定义资源定义（CRD）提供了以与内置Kubernetes资源相同的方式创建和管理TF作业的功能。 部署后，CRD可以配置TensorFlow job，允许用户专注于机器学习而不是基础设施。   

##### 2.3.1 创建TFJob部署定义
要部署上一步中描述的TensorFlow工作负载，Kubeflow需要TFJob定义。 在这种情况下，可以通过运行cat example.yaml来查看它:   

```
apiVersion: "kubeflow.org/v1alpha2"
kind: "TFJob"
metadata:
  name: "example-job"
spec:
  tfReplicaSpecs:
    Master:
      replicas: 1
      restartPolicy: Never
      template:
        spec:
          containers:
            - name: tensorflow
              image: gcr.io/tf-on-k8s-dogfood/tf_sample:dc944ff
    Worker:
      replicas: 1
      restartPolicy: Never
      template:
        spec:
          containers:
            - name: tensorflow
              image: gcr.io/tf-on-k8s-dogfood/tf_sample:dc944ff
    PS:
      replicas: 2
      restartPolicy: Never
      template:
        spec:
          containers:
            - name: tensorflow
              image: gcr.io/tf-on-k8s-dogfood/tf_sample:dc944ff
```
以上yaml定义了三个组件:    

Master: 每个job必须有一个master. Master将协调workers之间的训练操作的执行.    
Worker: 每个job可以有0到N个workers.
每个worker进程运行相同的模型，为参数服务器(Parameter Server)提供处理参数。  
PS: 每个job可以有0到N个参数服务器(Parameter Server)，
参数服务器使得用户可以将模型扩展到多台机器上。   

##### 2.3.2 部署TFJob
TFJob可以用以下命令来创建:    

```
# kubectl apply -f example.yaml
```
通过部署job，Kubernetes将调度工作负载以跨可用节点执行。 作为部署的一部分，Kubeflow将使用所需的设置配置TensorFlow，以允许不同的组件进行通信。   

##### 2.3.3 检查Job进度及处理结果
可以通过kubectl get tfjob查看TensorFlow作业的状态。 完成TensorFlow作业后，Master将标记为成功。 继续运行kubectl get tfjob命令以查看它何时完成。

Master负责协调作业的执行，并汇总结果。可以使用`kubectl get pods| grep
completed`列出已完成的工作负载。    

```
# kubectl get pods | grep Completed
example-job-master-0                                  0/1     Completed   0          7m36s
```
在此示例中，结果输出到STDOUT，可使用kubectl日志查看。    

以下命令将输出结果：

```
# kubectl logs $(kubectl get pods | grep Completed | tr -s ' ' | cut -d ' ' -f 1)
INFO:root:Tensorflow version: 1.3.0-rc2
INFO:root:Tensorflow git version: v1.3.0-rc1-27-g2784b1c
INFO:root:tf_config: {u'cluster': {u'worker': [u'example-job-worker-0.default.svc.cluster.local:2222'], u'ps': [u'example-job-ps-0.default.svc.cluster.local:2222', u'example-job-ps-1.default.svc.cluster.local:2222'], u'master': [u'example-job-master-0.default.svc.cluster.local:2222']}, u'task': {u'index': 0, u'type': u'master'}}
```
可以在master, worker以及parameter servers上看到工作负载的执行结果。

#### 2.4 访问Model Server
一旦训练完成，该模型可用于在新数据发布时对其进行预测。
通过使用Kubeflow，可以通过将作业部署到Kubernetes基础结构，从而使得Model
Server变得可用.    

##### 2.4.1 部署Trained Model Server
Kubeflow tf-serving提供了服务TensorFlow模型的模板。 这可以通过使用Ksonnet定制和部署，并根据您的模型定义参数。   

通过部署脚本已经部署完毕Model Server, 客户端可以链接并访问到trained data(训练好的数据), 可以查看pod信息:    

```
kubectl get pods

......

inception-v1-d66f8848-gq8hh                           1/1     Running     0          7m2s

......

```
上面的inception就是我们训练好的模型.    
##### 2.4.2 例:图像分类
在这个例子中，我们使用预先训练的Inception V3模型。 这是在ImageNet数据集上训练的架构。 ML任务是图像分类，而模型服务器及其客户端由Kubernetes处理。    

要使用已发布的模型，您需要设置客户端。 这可以通过与其他工作相同的方式实现。 用于部署客户端的YAML文件可以通过`cat ~/model-client-job.yaml`来查看。 要部署它，请使用以下命令：

```
kubectl apply -f ~/model-client-job.yaml
```
文件内容:    

```
# cat model-client-job.yaml 
apiVersion: batch/v1
kind: Job
metadata:
  name: model-client-job-katacoda
spec:
  template:
    metadata:
      name: model-client-job-katacoda
    spec:
      containers:
      - name: model-client-job-katacoda
        image: katacoda/tensorflow_serving:localimage
        imagePullPolicy: Never
        command:
        - /bin/bash
        - -c
        args:
        - /serving/bazel-bin/tensorflow_serving/example/inception_client
          --server=inception:9000 --image=/data/katacoda.jpg
      restartPolicy: Never
```
如需查看`model-client-job`运行的状态，则运行:

```
kubectl get pods
```

![/images/2019_06_03_12_17_53_896x451.jpg](/images/2019_06_03_12_17_53_896x451.jpg)


以下命令将输出图像分类的结果:    

```
kubectl logs $(kubectl get pods | grep Completed | tail -n1 |  tr -s ' ' | cut -d ' ' -f 1)
```

![/images/2019_06_03_12_18_15_552x736.jpg](/images/2019_06_03_12_18_15_552x736.jpg)

### 3. 部署pytorch
#### 3.1 部署PyTorch扩展
Kubeflow使用自定义资源定义（CRD）和运算符扩展了Kubernetes。 每个自定义资源都旨在支持机器学习工作负载的部署。 定义好资源后，Operator将处理部署请求。   

使用`kubectl get crd`命令可以查看到自定义的PyTorch自定义资源已被创建:    

```
# kubectl get crd
NAME                       CREATED AT
pytorchjobs.kubeflow.org   2019-06-03T04:19:22Z
tfjobs.kubeflow.org        2019-06-03T02:46:43Z
```
#### 3.2 部署PyTorch工作负载
使用打包好的容器镜像启动PyTorch，该镜像中已打包了分布式MNIST模型。模型的python代码可以在以下链接查看:     

[https://github.com/kubeflow/pytorch-operator/blob/9605eb6783e3549654082ea4b18a9cb0391e8548/examples/dist-mnist/dist_mnist.py](https://github.com/kubeflow/pytorch-operator/blob/9605eb6783e3549654082ea4b18a9cb0391e8548/examples/dist-mnist/dist_mnist.py)     

要部署该训练模型，我们需要创建一个PyTorch Job， PyTorch
Job中定义了需使用的容器镜像以及训练所需要启动的副本数,
我们用于启动训练模型的yaml定义文件如下所示:     

```
# cat pytorch_example.yaml 
apiVersion: "kubeflow.org/v1alpha1"
kind: "PyTorchJob"
metadata:
  name: "distributed-mnist"
spec:
  backend: "tcp"
  masterPort: "23456"
  replicaSpecs:
    - replicas: 1
      replicaType: MASTER
      template:
        spec:
          containers:
          - image: gcr.io/kubeflow-ci/pytorch-dist-mnist_test:1.0
            imagePullPolicy: IfNotPresent
            name: pytorch
          restartPolicy: OnFailure
    - replicas: 3
      replicaType: WORKER
      template:
        spec:
          containers:
          - image: gcr.io/kubeflow-ci/pytorch-dist-mnist_test:1.0
            imagePullPolicy: IfNotPresent
            name: pytorch
          restartPolicy: OnFailure
```
通过以下命令来创建训练任务:     

```
# kubectl create -f pytorch_example.yaml
```
Kubeflow PyTorch Operator和Kubernetes将安排工作负载并启动所需数量的副本。
您可以使用kubectl get pods -l pytorch_job_name = distributed-mnist查看状态,
使用此命令将看到一个master和3个worker被创建。     

#### 3.3 查看进度
训练应该运行大约10个epochs(时期)，并且在CPU群集上需要5-10分钟。可以使用以下命令检查日志以查看训练进度:    

```
PODNAME=$(kubectl get pods -l pytorch_job_name=distributed-mnist,task_index=0 -o name)
kubectl logs ${PODNAME}
```
输出类似于以下内容：

```
Downloading http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz
Downloading http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz
Downloading http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz
Downloading http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz
Processing...
Done!
Rank  0 , epoch  0 :  1.2745472780232237
Rank  0 , epoch  1 :  0.5743547164872765
```
您可以通过在节点上使用htop来了解训练如何利用所有的CPU内核。如果我们将其他节点添加到Kubernetes集群，则三个副本将分布在节点上并加快训练时间。    

随着训练的进行，可以使用Kubectl查看状态以及训练何时完成。通过以yaml输出作业，可以查看执行的内部细节。这包括状态。    

```
kubectl get -o yaml pytorchjobs distributed-mnist
kubectl get -o json pytorchjobs distributed-mnist  | jq .status
kubectl get -o json pytorchjobs distributed-mnist  | jq .status.state
```

训练结束后，结果如下所示：    

```
Downloading http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz
Downloading http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz
Downloading http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz
Downloading http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz
Processing...
Done!
Rank  0 , epoch  0 :  1.2745472780232237
Rank  0 , epoch  1 :  0.5743547164872765
Rank  0 , epoch  2 :  0.4351522875810737
Rank  0 , epoch  3 :  0.36984553888662536
Rank  0 , epoch  4 :  0.3216655456038045
Rank  0 , epoch  5 :  0.2951831894356813
Rank  0 , epoch  6 :  0.2750083558114448
Rank  0 , epoch  7 :  0.2595048323273659
Rank  0 , epoch  8 :  0.24862684973521526
Rank  0 , epoch  9 :  0.22692083941101393
```
可以看到，随着训练的进行，损失逐渐减小。    
