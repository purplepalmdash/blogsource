+++
title = "WorkingTipsOnUbuntuGlusterFS"
date = "2019-01-22T21:07:40+08:00"
description = "WorkingTipsOnUbuntuGlusterFS"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Install
In 4 nodes, install the latest glusterfs via:    

```
sudo add-apt-repository -y ppa:gluster/glusterfs-5
sudo apt-get -y update
sudo apt-get -y install glusterfs-server
sudo apt-get -y install thin-provisioning-tools
```

### Configure
See following steps for configurating the gluster in 4
nodes(10.48.129.101~104):    

```
root@gluster-1:/home/vagrant# gluster peer status
Number of Peers: 0
root@gluster-1:/home/vagrant# gluster pool list
UUID					Hostname 	State
628ce8f2-622c-4be3-92b0-d8e5241d01b8	localhost	Connected 
root@gluster-1:/home/vagrant# ifconfig eth1
eth1      Link encap:Ethernet  HWaddr 52:54:00:c8:74:01  
          inet addr:10.48.129.101  Bcast:10.48.129.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fec8:7401/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2456 errors:0 dropped:17 overruns:0 frame:0
          TX packets:103 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:126151 (126.1 KB)  TX bytes:10351 (10.3 KB)

root@gluster-1:/home/vagrant# gluster peer probe 10.48.129.102
peer probe: success. 
root@gluster-1:/home/vagrant# gluster peer probe 10.48.129.103
peer probe: success. 
root@gluster-1:/home/vagrant# gluster peer probe 10.48.129.104
peer probe: success. 
root@gluster-1:/home/vagrant# gluster peer status
Number of Peers: 3

Hostname: 10.48.129.102
Uuid: 52472b8f-b835-4038-b061-89229d825c42
State: Peer in Cluster (Connected)

Hostname: 10.48.129.103
Uuid: 46161901-954a-4ffe-9aec-a8c14ceded03
State: Peer in Cluster (Connected)

Hostname: 10.48.129.104
Uuid: a597a3ff-8c32-4a5a-b83d-1d25187afe31
State: Peer in Cluster (Connected)
```

After peer probe, the cluster is listed as following:    

```
root@gluster-1:/home/vagrant# gluster peer status
Number of Peers: 3

Hostname: 10.48.129.102
Uuid: 52472b8f-b835-4038-b061-89229d825c42
State: Peer in Cluster (Connected)

Hostname: 10.48.129.103
Uuid: 46161901-954a-4ffe-9aec-a8c14ceded03
State: Peer in Cluster (Connected)

Hostname: 10.48.129.104
Uuid: a597a3ff-8c32-4a5a-b83d-1d25187afe31
State: Peer in Cluster (Connected)
root@gluster-1:/home/vagrant# gluster pool list
UUID					Hostname     	State
52472b8f-b835-4038-b061-89229d825c42	10.48.129.102	Connected 
46161901-954a-4ffe-9aec-a8c14ceded03	10.48.129.103	Connected 
a597a3ff-8c32-4a5a-b83d-1d25187afe31	10.48.129.104	Connected 
628ce8f2-622c-4be3-92b0-d8e5241d01b8	localhost    	Connected 
```
### heketi
Download the heketi from:    

```
# wget https://github.com/heketi/heketi/releases/download/v8.0.0/heketi-v8.0.0.linux.amd64.tar.gz
# tar xzvf heketi-v8.0.0.linux.amd64.tar.gz
# cd heketi
# cp heketi heketi-cli /usr/bin
```
Configuration:    

```
# cd heketi
# cp heketi.json heketi.json.back
# vim heketi.json
# mkdir -p /etc/heketi
# cp heketi.json /etc/heketi
```
Your content is listed as following:     

```
......
#修改端口，防止端口冲突
  "port": "18080",
......
#允许认证
  "use_auth": true,
......
#admin用户的key改为adminkey
      "key": "adminkey"
......
#修改执行插件为ssh，并配置ssh的所需证书，注意要能对集群中的机器免密ssh登陆，使用ssh-copy-id把pub key拷到每台glusterfs服务器上
    "executor": "ssh",
    "sshexec": {
      "keyfile": "/root/.ssh/id_rsa",
      "user": "root",
      "port": "22",
      "fstab": "/etc/fstab"
    },
......
# 定义heketi数据库文件位置
    "db": "/var/lib/heketi/heketi.db"
......
#调整日志输出级别
    "loglevel" : "warning"
```

Now start the heketi server via:     

```
# heketi --config=/etc/heketi/heketi.json
```
Your server now is listening at `localhost:18080`, now use the cli for
creating the cluster and volume.    

```
# heketi-cli --server http://localhost:18080 --user admin --secret "adminkey" cluster list
# heketi-cli --server http://localhost:18080 --user admin --secret "adminkey" cluster create
# alias heketi-cli-admin='heketi-cli --server http://localhost:18080 --user admin --secret "adminkey"'
# heketi-cli-admin node add --cluster "360e5d504ff7cc18823d580bac55711a" --management-host-name 10.48.129.101 --storage-host-name 10.48.129.101 --zone 1
# heketi-cli-admin node add --cluster "360e5d504ff7cc18823d580bac55711a" --management-host-name 10.48.129.102 --storage-host-name 10.48.129.102 --zone 1
# heketi-cli-admin node add --cluster "360e5d504ff7cc18823d580bac55711a" --management-host-name 10.48.129.103 --storage-host-name 10.48.129.103 --zone 1
# heketi-cli-admin node add --cluster "360e5d504ff7cc18823d580bac55711a" --management-host-name 10.48.129.104 --storage-host-name 10.48.129.104 --zone 1
# heketi-cli-admin --json device add --name="/dev/vdb" --node "f787daa4ff84461b11b7f7d00645dfdc"
# heketi-cli-admin --json device add --name="/dev/vdb" --node "16f738ee9822fb79c7b321c0b7ed1792"
# heketi-cli-admin --json device add --name="/dev/vdb" --node "c6b1f0ccfe1ff76a245012108c44043f"
# heketi-cli-admin --json device add --name="/dev/vdb" --node "2e631bedcb6b58c4b82bba31082531f1"
# heketi-cli-admin volume list
# heketi-cli-admin volume create --size=10
```
After it finishes, your volume will be created.   
