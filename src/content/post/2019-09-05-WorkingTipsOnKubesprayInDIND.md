+++
title = "WorkingTipsOnKubesprayDIND"
date = "2019-09-05T10:53:18+08:00"
description = "WorkingTipsOnKubesprayDIND"
keywords = ["Linux"]
categories = ["Linux"]
+++
### vagrant machine
Create a vagrant machine with 8-core/10G memory:   

```
vagrant init generic/ubuntu1604
vagrant up
vagrant ssh
```
### steps
Prepare the environment:    

```
sudo apt-get update -y
sudo apt-get install -y python-pip git python3-pip
git clone xxxxxxx/kubespray
cd kubespray
export LC_ALL="en_US.UTF-8"
pip install -r requirements.txt 
pip3 install ruamel.yaml
sudo apt-get install     apt-transport-https     ca-certificates     curl     gnupg-agent     software-properties-common
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update -y &&  sudo apt-get install docker-ce docker-ce-cli containerd.io -y
cd contrib/dind
pip install -r requirements.txt
```

Deploy the dind cluster:    

```
sudo /home/vagrant/.local/bin//ansible-playbook -i hosts dind-cluster.yaml
rm -f inventory/local-dind/hosts.yml 
sudo CONFIG_FILE=${INVENTORY_DIR}/hosts.yml /tmp/kubespray.dind.inventory_builder.sh
sudo chown -R vagrant /home/vagrant/.ansible/
sudo docker exec kube-node1 apt-get install -y iputils-ping
/home/vagrant/.local/bin//ansible-playbook --become -e ansible_ssh_user=ubuntu -i ${INVENTORY_DIR}/hosts.yml cluster.yml --extra-vars @contrib/dind/kubespray-dind.yaml
```
