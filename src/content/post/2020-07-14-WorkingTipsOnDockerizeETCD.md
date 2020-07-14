+++
title= "WorkingTipsOnDockerizeETCD"
date = "2020-07-14T09:50:04+08:00"
description = "WorkingTipsOnDockerizeETCD"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Environment
Ubuntu 18.04.4, IP `10.137.149.2`, docker run single etcd instance.  

### cfssl
Install cfssl in ubuntu 18.04.4 via:    

```
# apt-get install -y golang-cfssl
```
Generate cert:    

```
# mkdir -p /opt/k8s/cert && cd /opt/k8s/cert
# echo '{"CN":"CA","key":{"algo":"rsa","size":2048}}' | cfssl gencert -initca - | cfssljson -bare ca -  #Generate a new key and cert from CSR
# echo '{"signing":{"default":{"expiry":"438000h","usages":["signing","key encipherment","server auth","client auth"]}}}' > ca-config.json  #配置ca-config文件
# export ADDRESS="10.137.149.2,hostA"
# export NAME=etcd-server
# echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
# mkdir /etc/kubernetes/cfssl/ -p && cd 
# cp etcd-server.csr etcd-server-key.pem etcd-server.pem /etc/kubernetes/cfssl/
# ls /etc/kubernetes/cfssl/
ca.pem  etcd-server.csr  etcd-server-key.pem  etcd-server.pem
```
### etcd file
use binary file downloaded from github:    

```
# wget https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz
# tar zxvf etcd-v3.3.11-linux-amd64.tar.gz
# cd etcd-v3.3.11-linux-amd64
# cp etcd etcdctl /usr/bin/
```

### systemd configuration
Create a etcd.service file:    

```
# vim /etc/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
# set GOMAXPROCS to number of processors
ExecStart=/usr/bin/etcd \
  --name ${ETCD_NAME} \
  --cert-file=/etc/kubernetes/cfssl/etcd-server.pem \
  --key-file=/etc/kubernetes/cfssl/etcd-server-key.pem \
  --peer-cert-file=/etc/kubernetes/cfssl/etcd-server.pem \
  --peer-key-file=/etc/kubernetes/cfssl/etcd-server-key.pem \
  --trusted-ca-file=/etc/kubernetes/cfssl/ca.pem \
  --peer-trusted-ca-file=/etc/kubernetes/cfssl/ca.pem \
  --initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls ${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
  --advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \
  --initial-cluster infra1=https://10.137.149.2:2380 \
  --initial-cluster-state new \
  --data-dir=${ETCD_DATA_DIR}

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
Configuration file:    

```
ETCD_NAME="--name node1"
ETCD_DATA_DIR="--data-dir /var/lib/etcd/default.etcd"
ETCD_INITIAL_ADVERTISE_PEER_URLS="--initial-advertise-peer-urls https://10.137.149.2:2380"
ETCD_LISTEN_PEER_URLS="--listen-peer-urls https://10.137.149.2:2380"
ETCD_LISTEN_CLIENT_URLS="--listen-client-urls https://10.137.149.2:2379"
ETCD_ADVERTISE_CLIENT_URLS="--advertise-client-urls https://10.137.149.2:2379"
ETCD_INITIAL_CLUSTER="--initial-cluster node1=https://10.137.149.2:2380"
ETCD_INITIAL_CLUSTER_STATE="--initial-cluster-state new"
ETCD_INITIAL_CLUSTER_TOKEN="--initial-cluster-token k8s-etcd"
ETCD_CERT_FILE="--cert-file /etc/kubernetes/cfssl/etcd-server.pem"
ETCD_KEY_FILE="--key-file /etc/kubernetes/cfssl/etcd-server-key.pem"
ETCD_TRUSTED_CA_FILE="--trusted-ca-file /etc/kubernetes/cfssl/ca.pem"
ETCD_CLIENT_CERT_AUTH="--client-cert-auth"
ETCD_PEER_CERT_FILE="--peer-cert-file /etc/kubernetes/cfssl/etcd-server.pem"
ETCD_PEER_KEY_FILE="--peer-key-file /etc/kubernetes/cfssl/etcd-server-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="--peer-trusted-ca-file /etc/kubernetes/cfssl/ca.pem"
ETCD_PEER_CLIENT_CERT_AUTH="--peer-client-cert-auth"
```
Start service via:    

```
# systemctl enable etcd
# systemctl start etcd
```

### Test
Via following commands:     

```
# ETCDCTL_API=3 /usr/bin/etcdctl --endpoints=https://10.137.149.2:2379 --cacert="/opt/k8s/cert/ca.pem" --cert="/opt/k8s/cert/etcd-server.pem" --key="/opt/k8s/cert/etcd-server-key.pem" member list
# ETCDCTL_API=3 /usr/bin/etcdctl --endpoints=https://10.137.149.2:2379 --cacert="/opt/k8s/cert/ca.pem" --cert="/opt/k8s/cert/etcd-server.pem" --key="/opt/k8s/cert/etcd-server-key.pem" version
# ETCDCTL_API=3 /usr/bin/etcdctl --endpoints=https://10.137.149.2:2379 --cacert="/opt/k8s/cert/ca.pem" --cert="/opt/k8s/cert/etcd-server.pem" --key="/opt/k8s/cert/etcd-server-key.pem" put foo bar
# ETCDCTL_API=3 /usr/bin/etcdctl --endpoints=https://10.137.149.2:2379 --cacert="/opt/k8s/cert/ca.pem" --cert="/opt/k8s/cert/etcd-server.pem" --key="/opt/k8s/cert/etcd-server-key.pem" get foo
```
