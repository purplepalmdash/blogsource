+++
title = "DockerNetworkPerformanceTest"
description = "DockerNetworkPerformanceTest"
date = "2017-07-13T16:51:35+08:00"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 测试环境

Docker常用的两种网络模式包括Bridge和Host模式，为测试这两种网络模式的性能，我们将创建以下的测试环境:    

* 192.192.192.89 - 运行Docker容器的服务器， CentOS 7.3.    
* 192.192.192.88 - 运行客户端的服务器, CentOS 7.3.    

两台服务器之间的物理网络为万兆以太网络。    
 
我们采用Iperf[http://software.es.net/iperf/](http://software.es.net/iperf/)来测量网络带宽，iperf非常简单，也拥有足够多的特性用于测试基本的性能指标。
在服务器端，我们需要一个运行iperf3的Docker容器。 Docker的版本为17.05-ce.    

测试将基于以下几个场景:
* 原始网络吞吐量
* 跨主机物理机到Docker(host模式)
* 跨主机物理机到Docker(Bridge模式)
* 同主机物理机到Docker(Bridge模式)
* 同主机Docker到Docker(Bridge模式-external)
* 同主机Docker到Docker(Bridge模式-internal)

### 原始网络吞吐量
首先，我们需要得到在没有任何Docker容器运行时的原始网络吞吐，在Server端运行:   

```
[root@192.192.192.89 ~]# iperf3 -s -p 5202
```
Client端运行:    

```
[root@192.192.192.88 ~]# iperf3 -c 192.192.192.89 -p 5202
```

运行测试后，服务器端和客户端都会返回诊断信息。我们暂时只关心其吞吐量:    

```
-----------------------------------------------------------
Server listening on 5202
-----------------------------------------------------------
Accepted connection from 192.192.192.88, port 39682
[  5] local 192.192.192.89 port 5202 connected to 192.192.192.88 port 39684
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  1.05 GBytes  9.05 Gbits/sec                  
[  5]   1.00-2.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   2.00-3.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   3.00-4.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   4.00-5.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   5.00-6.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   6.00-7.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   7.00-8.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   8.00-9.00   sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]   9.00-10.00  sec  1.10 GBytes  9.41 Gbits/sec                  
[  5]  10.00-10.04  sec  42.0 MBytes  9.39 Gbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-10.04  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-10.04  sec  11.0 GBytes  9.38 Gbits/sec                  receiver
-----------------------------------------------------------
Server listening on 5202
-----------------------------------------------------------
```
可以看到，在万兆交换机的网络场景下，物理机到物理机之间的网络带宽跑满了万兆交换机的极限.   

### 跨主机物理机到Docker(host模式)
在Docker中运行`iperf3`相当简单，在`hub.docker.com`可以找到大量的打包有iperf3的镜像，我们采用:    

```
# sudo docker pull networkstatic/iperf3
```
在服务器端启动侦听`5203`端口的docker实例:    

```
[root@192.192.192.89 ~]# docker run --net=host  -it --rm --name=iperf3-server networkstatic/iperf3 -s -p 5203
```
在Client端执行对应的修改，得到的结果为:    

```
[root@192.192.192.88 ~]# iperf3 -c 192.192.192.89 -p 5203
Connecting to host 192.192.192.89, port 5203
[  4] local 192.192.192.88 port 40326 connected to 192.192.192.89 port 5203
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  1.10 GBytes  9.43 Gbits/sec   20    625 KBytes       
[  4]   1.00-2.00   sec  1.10 GBytes  9.42 Gbits/sec    0    625 KBytes       
//.....
```
结果差不多相同: 9.40 Gbits/sec

### 跨主机物理机到Docker(Bridge模式)
更改为5204端口，这次使用的网络模式为`Bridge`模式:    

```
[root@192.192.192.89 ~]# docker run  -it --rm -p 5204:5204 --name=iperf3-server networkstatic/iperf3 -s -p 5204
```
在客户端不作任何修改，只更换远端端口为5204:    

```
[root@192.192.192.88 ~]# iperf3 -c 192.192.192.89 -p 5204
Connecting to host 192.192.192.89, port 5204
[  4] local 192.192.192.88 port 53936 connected to 192.192.192.89 port 5204
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  1.10 GBytes  9.44 Gbits/sec   15    669 KBytes       
[  4]   1.00-2.00   sec  1.10 GBytes  9.42 Gbits/sec    0    682 KBytes       
[  4]   2.00-3.00   sec  1.10 GBytes  9.42 Gbits/sec    0    691 KBytes 
```
可以看到，在Bridge模式下，吞吐量也跑满了万兆网络的极限.    

### 同主机物理机到Docker(Bridge模式)
在同一台主机上(192.192.192.89)上运行iperf，测试到Docker的吞吐量，沿用之前侦听5204的容器不变:    

```
[root@192.192.192.89 ~]# iperf3 -c 192.192.192.89 -p 5204
Connecting to host 192.192.192.89, port 5204
[  4] local 192.192.192.89 port 46720 connected to 192.192.192.89 port 5204
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  2.77 GBytes  23.8 Gbits/sec    0    274 KBytes       
[  4]   1.00-2.00   sec  2.75 GBytes  23.6 Gbits/sec    0    274 KBytes       
[  4]   2.00-3.00   sec  2.75 GBytes  23.6 Gbits/sec    0    277 KBytes       
```
在这种模式下，网络的吞吐量几乎三倍于万兆网络，这是因为从主机到Docker实例的网络通路会走本地的回环接口(lo-loopback)接口。    

### 同主机Docker到Docker(Bridge模式-external)
沿用侦听5204端口的容器不变，新启动一个容器，在其中运行iperf:    

```
# iperf3 -c 192.192.192.89 -p 5204
Connecting to host 192.192.192.89, port 5204
[  4] local 172.17.0.5 port 59574 connected to 192.192.192.89 port 5204
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  1.03 GBytes  8.84 Gbits/sec   91    228 KBytes       
[  4]   1.00-2.00   sec   955 MBytes  8.01 Gbits/sec    0    229 KBytes       
[  4]   2.00-3.00   sec  1.02 GBytes  8.80 Gbits/sec    0    230 KBytes       
[  4]   3.00-4.00   sec   767 MBytes  6.43 Gbits/sec    0    230 KBytes       
[  4]   4.00-5.00   sec   851 MBytes  7.14 Gbits/sec    0    230 KBytes       
```
可以看到，如果直接使用物理机的IP地址和端口，则吞吐需要同时使用Bridge模式下物理网卡的吞吐，
此时网卡的物理性能下降明显。     

### 同主机Docker到Docker(Bridge模式-internal)
为了避免使用物理机的IP地址带来的性能下降，直接使用容器内部的IP地址做iperf测试:    

```
# iperf3 -c 172.17.0.4 -p 5204
Accepted connection from 172.17.0.5, port 39516
[  5] local 172.17.0.4 port 5204 connected to 172.17.0.5 port 39518
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  2.39 GBytes  20.5 Gbits/sec                  
[  5]   1.00-2.00   sec  2.50 GBytes  21.5 Gbits/sec                  
[  5]   2.00-3.00   sec  2.50 GBytes  21.5 Gbits/sec 
```
可以看到，在这种模式下，容器之间的通信还是基于lo(loopback)接口来做的，几乎三倍于万兆交换机的峰值速度。    

### 结论
各次测试的对比数据整理如下:    

```
| 物理机-物理机                           | 9.40 Gbit/sec    | 100%  |
| 跨物理机到Docker(host模式网络)          | 9.40 Gbit/sec    | 100%  |
| 跨物理机到Docker(Bridge模式网络)        | 9.40 Gbit/sec    | 100%  |
| 同主机内到Docker(Bridge模式网络)        | 23.8 Gbit/sec    | 250%  |
| 同主机Docker到Docker(Bridge模式-ex)     | 8.00 Gbit/sec    |  85%  |
| 同主机Docker到Docker(Bridge模式-int)    | 21.00 Gbit/sec   | 220%  |
```

结论： 在Docker运行环境中，网络的吞吐量近似于本地网络IO，基本上不会有性能损耗。需要特别注意的是，一定要避免同主机中的Docker实例彼此使用物理机IP/端口进行通信，那样会带来性能的明显下降。    
