+++
title = "TipsOnK8SScheduler"
date = "2019-04-12T23:24:14+08:00"
description = "TipsOnK8SScheduler"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 问题
文章来源:   
[https://blog.csdn.net/horsefoot/article/details/51577402](https://blog.csdn.net/horsefoot/article/details/51577402)    

![./images/2019_04_12_23_25_30_1078x550.jpg](./images/2019_04_12_23_25_30_1078x550.jpg)    

印象中我没有见过K8S中有这样的调度策略，那这个博客的提法是从何而来？    

### 追溯
Google关键字，看到这个比较类似：    

![/images/2019_04_12_23_29_24_763x231.jpg](/images/2019_04_12_23_29_24_763x231.jpg)

fabric8(一个基于k8s的CI/CD平台)里的条目, 时间维度是16年3月：    

![/images/2019_04_12_23_30_49_749x248.jpg](/images/2019_04_12_23_30_49_749x248.jpg)

内中条目:    

![/images/2019_04_12_23_31_52_900x349.jpg](/images/2019_04_12_23_31_52_900x349.jpg)

看来k8s的proposal文档中确实有过这个条目，继续用不同关键字搜索,
找到一个相关issue:    

![/images/2019_04_12_23_33_13_648x245.jpg](/images/2019_04_12_23_33_13_648x245.jpg)

点进去阅读此issue:    

![/images/2019_04_12_23_36_41_756x610.jpg](/images/2019_04_12_23_36_41_756x610.jpg)

半成品状态：   

![/images/2019_04_12_23_39_19_627x477.jpg](/images/2019_04_12_23_39_19_627x477.jpg)

issue被re-assign给另外的开发者, 最终被关闭：    

![/images/2019_04_12_23_43_03_783x432.jpg](/images/2019_04_12_23_43_03_783x432.jpg)

回到其父issue:   


问题提出:    

![/images/2019_04_13_00_36_30_695x251.jpg](/images/2019_04_13_00_36_30_695x251.jpg)

```
当预测错误时，必须有反馈。特别是，我认为重要的是，超过其请求的Pod（由于不正确的初始预测）比其请求下的其他pod更可能被杀死。这样，“设置初始限制”系统的故障似乎会影响特定的pod，而不是随机的pod，这使得诊断非常困难。

一种方法是使系统OOM情况下的终止概率与请求量成比例。 @AnanyaKumar @vishh目前的实施有没有这个属性？
```
![/images/2019_04_13_00_23_35_775x634.jpg](/images/2019_04_13_00_23_35_775x634.jpg)


以上讨论解释了为何设置初始资源用量会带来更多问题：    

```
@erictune我看到你的观点，我同意设置限制会解决你的情况。 另一方面，我可以想象一些其他情况，它将带来更多的麻烦。 特别是当预先估计错误的时候，用户会观察到调度程序将意外杀死他的容器时。 因此，对资源的分配，采用设置限制的方式会带来更高的可靠性，我们无法从一开始就保证它。

我认为每个人都同意设置请求(Request)应该改善整体体验，这可能不适用于设置限制(Limits)。 从长远来看，我们肯定想要设置两者，但我只会在第一个版本中设置Request（可能与v1.1不同），从用户收集一些反馈，然后在我们调整算法后最终添加设置限制。

@vishh如果有两个容器超出他们的要求：哪一个将被杀死？ 超过请求'更多'或随机的一个？
```

当然，后面有一系列原因导致了`Vertical pod auto-sizer`这个特性的开发被终止。   

回到《从kubernetes看如何设计超大规模资源调度系统》一文的提法来看，作者可能当时仅仅是看了Kubernetes社区开发中的一个开发特性的建议文档，就认为Kubernetes拥有了"预测资源需求量"的功能。事实上从K8s社区的讨论来看，"预测资源需求"只是为了实现"vertical pod auto-sizer"，而这一特性一直到18年底才进入到alpha阶段，vpa特性的实现也不再是依靠16年时定义而实现。    

作者写该文章时，根据当时文档做出的实现还依赖于influxdb来收集指标，而influxdb+heapster应该是18年就被metric
server所替代。当前处于alpha阶段的vpa特性也是依赖于metric server而实现的。    

含有文章中提法的代码目录于v1.1版中被引入，至v1.11版被删除。   

![/images/2019_04_13_00_55_36_602x320.jpg](/images/2019_04_13_00_55_36_602x320.jpg)

v1.10版后被从主线删除：    

![/images/2019_04_13_00_56_54_564x542.jpg](/images/2019_04_13_00_56_54_564x542.jpg)

### 总结
从对issue的跟踪来看，应该是作者理解错了。预先估计资源用量从来不是Kubernetes定义资源需求的正确方法。   

官方的正确参考资料如下：    

[https://kubernetes.io/zh/docs/tasks/administer-cluster/cpu-memory-limit/](https://kubernetes.io/zh/docs/tasks/administer-cluster/cpu-memory-limit/)    
如果需要在生产环境中优化资源的使用，可以参考：   

[https://www.yangcs.net/posts/optimizing-kubernetes-resource-allocation-production/](https://www.yangcs.net/posts/optimizing-kubernetes-resource-allocation-production/)   

### 参考
Vertical pod auto-sizer, issue #10782:   

[https://github.com/kubernetes/kubernetes/issues/10782](https://github.com/kubernetes/kubernetes/issues/10782) 

Setting Initial Resources, issue #12149:    

[https://github.com/kubernetes/kubernetes/issues/12149](https://github.com/kubernetes/kubernetes/issues/12149)    

initial-resources.md:    

[https://github.com/fabric8io/kansible/blob/master/vendor/k8s.io/kubernetes/docs/proposals/initial-resources.md](https://github.com/fabric8io/kansible/blob/master/vendor/k8s.io/kubernetes/docs/proposals/initial-resources.md)     
