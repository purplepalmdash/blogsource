+++
title = "kismatic110tips"
date = "2018-04-18T16:38:20+08:00"
description = "kismatic110tips"
keywords = ["Linux"]
categories = ["Technology"]
+++
### preparation
Deployment machine, Download the packages:    

```
# mkdir deploy
# cd deploy
# wget https://github.com/apprenda/kismatic/releases/download/v1.10.0/kismatic-v1.10.0-linux-amd64.tar.gz
# git clone https://github.com/apprenda/kismatic.git
# tar xzvf *.tar.gz;
# ls 
ansible  helm  kismatic  kismatic-master  kismatic-master.zip  kismatic-v1.10.0-linux-amd64.tar.gz  kubectl  provision
```
Target node(all-in-one), install python-pip, shadowsocks, redsocks, gcc, etc, for acrossing the fucking GFW!  


### plan
plan the cluster

```
./kismatic install plan
Plan your Kubernetes cluster:
=> Number of etcd nodes [3]: 1
=> Number of master nodes [2]: 1
=> Number of worker nodes [3]: 1
=> Number of ingress nodes (optional, set to 0 if not required) [2]: 0
=> Number of storage nodes (optional, set to 0 if not required) [0]: 0
=> Number of existing files or directories to be copied [0]: 0

Generating installation plan file template with: 
- 1 etcd nodes
- 1 master nodes
- 1 worker nodes
- 0 ingress nodes
- 0 storage nodes
- 0 files

Wrote plan file template to "kismatic-cluster.yaml"
Edit the plan file to further describe your cluster. Once ready, execute the "install validate" command to proceed.
```
An empty `kismatic-cluster.yaml` will be generated, later we will edit it.  

### validate
validate with detailed information:    

```
./kismatic install validate -o raw
```
Error:    

```
ansible/bin/ansible-playbook -i ansible/inventory.ini -s ansible/playbooks/preflight.yaml --extra-vars @ansible/clustercatalog.yaml -vvvv
Traceback (most recent call last):
  File "ansible/bin/ansible-playbook", line 36, in <module>
    import shutil
  File "/usr/lib/python3.6/shutil.py", line 10, in <module>
    import fnmatch
  File "/usr/lib/python3.6/fnmatch.py", line 14, in <module>
    import re
  File "/usr/lib/python3.6/re.py", line 142, in <module>
    class RegexFlag(enum.IntFlag):
AttributeError: module 'enum' has no attribute 'IntFlag'
error running playbook: error running ansible: exit status 1
```
Seems because the python is python3 rather than python2.  

Edit the python definition:    

```
# vim ansible/bin/ansible-playbook
    #!/usr/bin/python2
``` 
Then your validation will be OK.   

### install apply
Via following command:    

```
# ./kismatic install apply
```

