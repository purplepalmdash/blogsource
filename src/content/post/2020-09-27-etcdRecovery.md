+++
title= "etcdRecovery"
date = "2020-09-27T16:42:14+08:00"
description = "etcdRecovery"
keywords = ["Technology"]
categories = ["Technology"]
+++

主节点操作: 

```
root@newnode-1:/home/test# ETCDCTL_API=3 etcdctl --endpoints=https://192.168.122.21:2379 --cacert="/etc/ssl/etcd/ssl/ca.pem" --cert="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" member list
4047613ce64ac480, started, etcd2, https://192.168.122.58:2380, https://192.168.122.58:2379
ac76e9faf75cf70f, started, etcd3, https://192.168.122.75:2380, https://192.168.122.75:2379
e99611c964d08e01, started, etcd1, https://192.168.122.21:2380, https://192.168.122.21:2379
root@newnode-1:/home/test# ETCDCTL_API=2 etcdctl --endpoints=https://192.168.122.21:2379 --ca-file="/etc/ssl/etcd/ssl/ca.pem" --cert-file="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key-file="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" cluster-health
member 4047613ce64ac480 is healthy: got healthy result from https://192.168.122.58:2379
failed to check the health of member ac76e9faf75cf70f on https://192.168.122.75:2379: Get https://192.168.122.75:2379/health: dial tcp 192.168.122.75:2379: connect: connection refused
member ac76e9faf75cf70f is unreachable: [https://192.168.122.75:2379] are all unreachable
member e99611c964d08e01 is healthy: got healthy result from https://192.168.122.21:2379
cluster is degraded
```

删除问题节点: 

```
root@newnode-1:/home/test# ETCDCTL_API=2 etcdctl --endpoints=https://192.168.122.21:2379 --ca-file="/etc/ssl/etcd/ssl/ca.pem" --cert-file="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key-file="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" member remove ac76e9faf75cf70f
Removed member ac76e9faf75cf70f from cluster
root@newnode-1:/home/test# ETCDCTL_API=2 etcdctl --endpoints=https://192.168.122.21:2379 --ca-file="/etc/ssl/etcd/ssl/ca.pem" --cert-file="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key-file="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" member list
4047613ce64ac480: name=etcd2 peerURLs=https://192.168.122.58:2380 clientURLs=https://192.168.122.58:2379 isLeader=true
e99611c964d08e01: name=etcd1 peerURLs=https://192.168.122.21:2380 clientURLs=https://192.168.122.21:2379 isLeader=false
```

问题节点上操作:

```
systemctl stop etcd
mv /var/lib/etcd /var/lib/etcd.back
mkdir /var/lib/etcd
systemctl start etcd
```

新增: 

```
root@newnode-1:/home/test# ETCDCTL_API=2 etcdctl --endpoints=https://192.168.122.21:2379 --ca-file="/etc/ssl/etcd/ssl/ca.pem" --cert-file="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key-file="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" member add etcd3 https://192.168.122.75:2380
Added member named etcd3 with ID 318e07d1cc0d3933 to cluster

ETCD_NAME="etcd3"
ETCD_INITIAL_CLUSTER="etcd3=https://192.168.122.75:2380,etcd2=https://192.168.122.58:2380,etcd1=https://192.168.122.21:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
root@newnode-1:/home/test# ETCDCTL_API=2 etcdctl --endpoints=https://192.168.122.21:2379 --ca-file="/etc/ssl/etcd/ssl/ca.pem" --cert-file="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key-file="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" member list
318e07d1cc0d3933[unstarted]: peerURLs=https://192.168.122.75:2380
4047613ce64ac480: name=etcd2 peerURLs=https://192.168.122.58:2380 clientURLs=https://192.168.122.58:2379 isLeader=true
e99611c964d08e01: name=etcd1 peerURLs=https://192.168.122.21:2380 clientURLs=https://192.168.122.21:2379 isLeader=false
```

如果是unstarted 状态，则到有问题节点:

```
systemctl stop etcd
rm -rf /var/lib/etcd/member
systemctl start etcd
```

回到主节点， 观察集群状态是否回复成功

```
root@newnode-1:/home/test# ETCDCTL_API=2 etcdctl --endpoints=https://192.168.122.21:2379 --ca-file="/etc/ssl/etcd/ssl/ca.pem" --cert-file="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key-file="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" member list
4047613ce64ac480: name=etcd2 peerURLs=https://192.168.122.58:2380 clientURLs=https://192.168.122.58:2379 isLeader=true
531c8ba1dbabce70: name=etcd3 peerURLs=https://192.168.122.75:2380 clientURLs=https://192.168.122.75:2379 isLeader=false
e99611c964d08e01: name=etcd1 peerURLs=https://192.168.122.21:2380 clientURLs=https://192.168.122.21:2379 isLeader=false
root@newnode-1:/home/test# ETCDCTL_API=2 etcdctl --endpoints=https://192.168.122.21:2379 --ca-file="/etc/ssl/etcd/ssl/ca.pem" --cert-file="/etc/ssl/etcd/ssl/member-newnode-1.pem" --key-file="/etc/ssl/etcd/ssl/member-newnode-1-key.pem" cluster-health
member 4047613ce64ac480 is healthy: got healthy result from https://192.168.122.58:2379
member 531c8ba1dbabce70 is healthy: got healthy result from https://192.168.122.75:2379
member e99611c964d08e01 is healthy: got healthy result from https://192.168.122.21:2379
cluster is healthy
```
