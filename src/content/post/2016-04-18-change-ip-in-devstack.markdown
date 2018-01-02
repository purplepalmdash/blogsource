---
categories: ["Virtualization"]
comments: true
date: 2016-04-18T12:15:35Z
title: Change IP In DevStack
url: /2016/04/18/change-ip-in-devstack/
---

### Tips
After changing the IP Address of the DevStack machine, do following for re-installing
the envs:    

```
$ ssh stack@Your_IP
stack@packer-PerforceTest:~$ pwd
/opt/stack
stack@packer-PerforceTest:~$ cd devstack/
stack@packer-PerforceTest:~/devstack$ ./unstack.sh && ./stack.sh 
```

Now visiting your `http://Your_New_IP/dashboard` you will got the openstack dashborad.   
