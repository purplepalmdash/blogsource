+++
title = "WorkingTipsOnOfflineOpenshift"
date = "2019-08-12T09:55:35+08:00"
description = "WorkingTipsOnOfflineOpenshift"
keywords = ["Linux"]
categories = ["Technology"]
+++
### OS Preparation
Centos 7.6 OS, installed via:    

```
CentOS-7-x86_64-Everything-1810.iso
```
Download the source code from:    

```
https://gitee.com/xhua/OpenshiftOneClick
```
Corresponding docker images:     

```
docker.io/redis:5
docker.io/openshift/origin-node:v3.11.0
docker.io/openshift/origin-control-plane:v3.11.0
docker.io/openshift/origin-haproxy-router:v3.11.0
docker.io/openshift/origin-deployer:v3.11.0
docker.io/openshift/origin-pod:v3.11.0
docker.io/rabbitmq:3.7-management
docker.io/mongo:4.1
docker.io/memcached:1.5
quay.io/kubevirt/kubevirt-web-ui-operator:latest
docker.io/xhuaustc/openldap-2441-centos7:latest
quay.io/kubevirt/kubevirt-web-ui:v2.0.0
docker.io/perconalab/pxc-openshift:latest
docker.io/tomcat:8.5-alpine
docker.io/centos/postgresql-95-centos7:latest
docker.io/centos/mysql-57-centos7:latest
docker.io/centos/nginx-112-centos7:latest
docker.io/curiouser/dubbo_zookeeper:v1
docker.io/xhuaustc/logstash:6.6.1
docker.io/xhuaustc/kibana:6.6.1
docker.io/xhuaustc/elasticsearch:6.6.1
docker.io/openshift/jenkins-2-centos7:latest
docker.io/openshift/origin-docker-registry:v3.11.0
docker.io/openshift/jenkins-agent-maven-35-centos7:v4.0
docker.io/openshift/origin-console:v3.11.0
docker.io/sonatype/nexus3:3.14.0
docker.io/gitlab/gitlab-ce:11.4.0-ce.0
docker.io/openshift/origin-web-console:v3.11.0
docker.io/cockpit/kubernetes:latest
docker.io/xhuaustc/apolloportal:latest
docker.io/xhuaustc/apolloconfigadmin:latest
docker.io/xhuaustc/nfs-client-provisioner:latest
docker.io/blackcater/easy-mock:1.6.0
docker.io/perconalab/proxysql-openshift:0.5
docker.io/xhuaustc/selenium:3
docker.io/xhuaustc/zalenium:3
docker.io/xhuaustc/etcd:v3.2.22
docker.io/openshiftdemos/gogs:0.11.34
docker.io/openshiftdemos/sonarqube:6.7
docker.io/xhuaustc/openshift-kafka:latest
docker.io/redis:3.2.3-alpine
docker.io/kubevirt/virt-api:v0.19.0
docker.io/kubevirt/virt-controller:v0.19.0
docker.io/kubevirt/virt-handler:v0.19.0
docker.io/kubevirt/virt-operator:v0.19.0
```
### Servers
### rpm server
ISO as a rpm server.    

offline iso rpm server.    

```
# vim files/all.repo
[openshift]
name=openshift
baseurl=http://192.192.189.1/ocrpmpkgs/
enabled=1
gpgcheck=0

[openshift1]
name=openshift1
baseurl=http://192.192.189.1:8080
enabled=1
gpgcheck=0
```

#### Simple https server
Create a new folder and generate pem files under this folder:    

```
# openssl req -new -x509 -keyout server.pem -out server.pem -days 365 -nodes
Common name: ssl.xxxx.com
```
If you set `ssl.xxxx.com`, then you visit this website via `https://ssl.xxxx.com:4443/index.html`.    

Write a simple python file for serving https:    

```
# vi simple-https-server.py
import BaseHTTPServer, SimpleHTTPServer
import ssl
httpd = BaseHTTPServer.HTTPServer(('localhost', 4443), SimpleHTTPServer.SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket (httpd.socket, certfile='./server.pem', server_side=True)
httpd.serve_forever()
# sudo python simple-https-server.py
```
Server folder content:    

```
# ls
allinone-webconsole.css  apollo.png  easymock.png  kafka.png  nexus3.png    pxc.png     simple-https-server.py  zalenium.png
allinone-webconsole.js   dubbo.png   gogs.png      kelk.png   openldap.png  server.pem  sonarqube.svg
# cat allinone-webconsole.css 
.icon-gogs{
  background-image: url(https://ssl.xxxx.com:4443/gogs.png);
  width: 50px;
  height: 50px;
  background-size: 100% 100%;
}
.icon-sonarqube{
  background-image: url(https://ssl.xxxx.com:4443/sonarqube.svg);
  width: 80px;
  height: 50px;
  background-size: 100% 100%;
}

```
#### Using Simple https server
Add customized domain name into `/etc/hosts`:    

```
# vim /etc/hosts
192.192.189.1	ssl.xxxx.com
```
Add `server.pem` into the client system:     

```
# yum install -y ca-certificates
# update-ca-trust force-enable
# cp server.pem /etc/pki/ca-trust/source/anchors/ssl.xxxx.com.pem
# update-ca-trust
```

### Deployment
Two nodes, run following scripts first:     

```
#!/bin/bash

setenforce 1
selinux=$(getenforce)
if [ "$selinux" != Enforcing ]
then
	echo "Please setlinux Enforcing"
	exit 10
fi

cat >/etc/sysctl.d/99-elasticsearch.conf <<EOF
vm.max_map_count = 262144
EOF
sysctl vm.max_map_count=262144

export CHANGEREPO=true
if [ $CHANGEREPO == true -a ! -d /etc/yum.repos.d/back ]
then
    cd /etc/yum.repos.d/; mkdir -p back; mv -f *.repo back/; cd -
    cp files/all.repo /etc/yum.repos.d/
    yum clean all
fi


current_path=`pwd`
yum localinstall tools/ansible-2.6.5-1.el7.ans.noarch.rpm -y
ansible-playbook playbook.yml --skip-tags after_task
cd $current_path/openshift-ansible-playbook
ansible-playbook playbooks/prerequisites.yml
```
Configuration of Master node's `config.yml`:    

```
---
CHANGEREPO: true
HOSTNAME: os311.test.it.example.com

```

Configuration of Worker node's `config.yml`:    

```
---
CHANGEREPO: true
HOSTNAME: os312.test.it.example.com
```
Then add following lines into `/etc/hosts`:     

```
192.192.189.128 os311.test.it.example.com
192.192.189.129 os312.test.it.example.com
192.192.189.1	ssl.xxxx.com
```
Then on master node, replace the `/etc/ansible/hosts` with our pre-defined one:      

```
.....
openshift_web_console_extension_script_urls=["https://ssl.xxxx.com:4443/allinone-webconsole.js"]
openshift_web_console_extension_stylesheet_urls=["https://ssl.xxxx.com:4443/allinone-webconsole.css"]

......
openshift_disable_check=memory_availability,disk_availability,package_availability,package_update,docker_image_availability,docker_storage_driver,docker_storage,package_version

.......
openshift_node_groups=[{'name': 'node-config-all-in-one', 'labels': ['node-role.kubernetes.io/master=true', 'node-role.kubernetes.io/infra=true']}, {'name': 'node-config-compute', 'labels': ['node-role.kubernetes.io/compute=true']}]
.......
[masters]
os311.test.it.example.com
[etcd]
os311.test.it.example.com

[nfs]
os311.test.it.example.com

[nodes]
os311.test.it.example.com openshift_node_group_name="node-config-all-in-one"
os312.test.it.example.com openshift_node_group_name='node-config-compute'
```
Now run deployment:    

```
current_path=`pwd`
cd $current_path/openshift-ansible-playbook
ansible-playbook playbooks/prerequisites.yml

ansible-playbook -vvvv playbooks/deploy_cluster.yml
oc adm policy add-cluster-role-to-user cluster-admin admin

cd $current_path

ansible-playbook playbook.yml --tags install_nfs
ansible-playbook playbook.yml --tags after_task
```

After deployment, check the status via:      

```
[root@os311 OpenshiftOneClick]# oc get nodes
NAME                        STATUS    ROLES          AGE       VERSION
os311.test.it.example.com   Ready     infra,master   3d        v1.11.0+d4cacc0
os312.test.it.example.com   Ready     compute        3d        v1.11.0+d4cacc0
```
### kube-virt
via following steps, deploy kubevirt:      

```
# kubectl apply -f kubevirt-operator.yaml
# kubectl apply -f kubevirt-cr.yaml
```
deploy ui:      

```
# cd web-ui-operator-master
# oc new-project kubevirt-web-ui
# cd deploy
# oc apply -f service_account.yaml
# oc apply -f role.yaml
# oc apply -f role_binding.yaml
# oc create -f crds/kubevirt_v1alpha1_kwebui_crd.yaml
# oc apply -f operator.yaml
# oc apply -f deploy/crds/kubevirt_v1alpha1_kwebui_cr.yaml
```

### DNS setting
By following steps:     

```
# vim /etc/dnsmasq.d/origin-dns.conf
address=/os311.test.it.example.com/192.192.189.128
# systemctl daemon-reload
# systemctl restart dnsmasq
```

### Create vm
The definition files should be modified into:    

```
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachine
metadata:
  name: testvm
spec:
  running: false
  template:
    metadata:
      labels: 
        kubevirt.io/size: small
        kubevirt.io/domain: testvm
    spec:
      domain:
        devices:
          disks:
            - name: containerdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            bridge: {}
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: containerdisk
          containerDisk:
            image: kubevirt/cirros-registry-disk-demo
            imagePullPolicy: IfNotPresent
        - name: cloudinitdisk
          cloudInitNoCloud:
            userDataBase64: SGkuXG4=
```
Thus we could launch the vms, notice we have to pull the images manually:    

```
# sudo docker pull kubevirt/cirros-registry-disk-demo
# sudo docker pull index.docker.io/kubevirt/virt-launcher:v0.19.0
# sudo docker pull kubevirt/virt-launcher:v0.19.0
```
