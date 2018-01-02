+++
categories = ["Virtualization"]
date = "2016-12-22T18:02:47+08:00"
description = "CoreOS tips"
keywords = ["Virtualization"]
title = "CoreOSTips"

+++
### Add additional ssh keys
Adding new keys into the deployed system:    

```
# echo 'ssh-rsa AAAAB3Nza.......  key@host' | update-ssh-keys -a core
```
### Write files
Take `/etc/environment` file for example:    

```
core@coreos1 ~ $ cat /usr/share/oem/cloud-config.yml 
#cloud-config
write_files:
    - path: /etc/environment
      permissions: 0644
      content: |
        COREOS_PUBLIC_IPV4=172.17.8.201
        COREOS_PRIVATE_IPV4=172.17.8.201
```
### Add User
Also add in the file `/usr/share/oem/cloud-config.yml`, like following:    

```
users:
  - name: "dash"
    passwd: "xxxxxxxxxxxxxxxxxx"
    groups:
      - "sudo"
      - "docker"
    ssh-authorized-keys:
      - "ssh-rsa ADD ME"
```
Password could be generated via `openssl -1 "YourPasswd"`

### Use Ansible
Install via:    

```
$ sudo ansible-galaxy install defunctzombie.coreos-bootstrap
$ sudo vim /etc/ansible/roles/defunctzombie.coreos-bootstrap/files/bootstrap.sh
if [[ -e $HOME/pypy-5.6-linux_x86_64-portable.tar.bz2 ]]; then
  tar -xjf $HOME/pypy-5.6-linux_x86_64-portable.tar.bz2
  #rm -rf $HOME/pypy-$PYPY_VERSION-linux64.tar.bz2
else
  wget -O - https://bitbucket.org/pypy/pypy/downloads/pypy-$PYPY_VERSION-linux64.tar.bz2 |tar -xjf -
fi

rm -rf pypy
mv -n pypy-5.6-linux_x86_64-portable pypy
```
Cause the old version 5.1.0 will have problems, we replace it with 5.6.    

Create playbook and inventory file:    

```
$ vim site.yml 
- hosts: coreos
  gather_facts: False
  roles:
    - defunctzombie.coreos-bootstrap
$ vim inventory
[coreos]
172.17.8.221
172.17.8.222
172.17.8.223

[coreos:vars]
ansible_ssh_user=core
ansible_python_interpreter=/home/core/bin/python
$ ansible-playbook -i inventory site.yml
$ ansible -i inventory all -m ping
```
