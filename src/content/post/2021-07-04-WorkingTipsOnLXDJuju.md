+++
title= "WorkingTipsOnLXDJuju"
date = "2021-07-04T15:32:39+08:00"
description = "WorkingTipsOnLXDJuju"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Commands
Bootstrap via:    

```
test@edge5:~$ juju bootstrap localhost overlord
Creating Juju controller "overlord" on localhost/localhost
Looking for packaged Juju agent version 2.9.5 for amd64
Located Juju agent version 2.9.5-ubuntu-amd64 at https://streams.canonical.com/juju/tools/agent/2.9.5/juju-2.9.5-ubuntu-amd64.tgz
To configure your system to better support LXD containers, please see: https://github.com/lxc/lxd/blob/master/doc/production-setup.md
Launching controller instance(s) on localhost/localhost...
 - juju-55e209-0 (arch=amd64)                 
Installing Juju agent on bootstrap instance
Fetching Juju Dashboard 0.7.1
Waiting for address
Attempting to connect to 10.53.118.136:22
Connected to 10.53.118.136
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 10.53.118.136 to verify accessibility...

Bootstrap complete, controller "overlord" is now available
Controller machines are in the "controller" model
Initial model "default" added
```
Verify bootstrap status:    

```
test@edge5:~$ lxc ls
+---------------+---------+----------------------+------+-----------+-----------+
|     NAME      |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+---------------+---------+----------------------+------+-----------+-----------+
| juju-55e209-0 | RUNNING | 10.53.118.136 (eth0) |      | CONTAINER | 0         |
+---------------+---------+----------------------+------+-----------+-----------+
test@edge5:~$ juju status
Model    Controller  Cloud/Region         Version  SLA          Timestamp
default  overlord    localhost/localhost  2.9.5    unsupported  15:32:12+08:00

Model "admin/default" is empty.
```
### microk8s
Deploy microk8s via:    

```
$ juju deploy -n3 cs:~pjdc/microk8s
Located charm "microk8s" in charm-store, revision 24
Deploying "microk8s" from charm-store charm "microk8s", revision 24 in channel stable
```
View juju status:    

```
test@edge5:~$ juju status
Model    Controller  Cloud/Region         Version  SLA          Timestamp
default  overlord    localhost/localhost  2.9.5    unsupported  15:35:54+08:00

App       Version  Status   Scale  Charm     Store       Channel  Rev  OS      Message
microk8s           waiting    0/3  microk8s  charmstore  stable    24  ubuntu  waiting for machine

Unit        Workload  Agent       Machine  Public address  Ports  Message
microk8s/0  waiting   allocating  0        10.53.118.110          waiting for machine
microk8s/1  waiting   allocating  1        10.53.118.99           waiting for machine
microk8s/2  waiting   allocating  2        10.53.118.115          waiting for machine

Machine  State    DNS            Inst id        Series  AZ  Message
0        pending  10.53.118.110  juju-585d2d-0  focal       Running
1        pending  10.53.118.99   juju-585d2d-1  focal       Running
2        pending  10.53.118.115  juju-585d2d-2  focal       Running
```
Until succeed:    

```
test@edge5:~$ juju status
Model    Controller  Cloud/Region         Version  SLA          Timestamp
default  overlord    localhost/localhost  2.9.5    unsupported  15:49:48+08:00

App       Version  Status  Scale  Charm     Store       Channel  Rev  OS      Message
microk8s           active      3  microk8s  charmstore  stable    24  ubuntu  

Unit         Workload  Agent  Machine  Public address  Ports                     Message
microk8s/0*  active    idle   0        10.53.118.110   80/tcp,443/tcp,16443/tcp  
microk8s/1   active    idle   1        10.53.118.99    80/tcp,443/tcp,16443/tcp  
microk8s/2   active    idle   2        10.53.118.115   80/tcp,443/tcp,16443/tcp  

Machine  State    DNS            Inst id        Series  AZ  Message
0        started  10.53.118.110  juju-585d2d-0  focal       Running
1        started  10.53.118.99   juju-585d2d-1  focal       Running
2        started  10.53.118.115  juju-585d2d-2  focal       Running
```

### own cloud
Ip and hostname listed as:    

```
192.168.89.6	edge5
192.168.89.7	edge6
192.168.89.8	edge7
192.168.89.9	edge8
192.168.89.10	edge9
```
Added :     

```
test@edge5:~$ ssh-copy-id test@192.168.89.7
test@edge5:~$ juju add-cloud                                                                                                                                  
This operation can be applied to both a copy on this client and to the one on a controller.
No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.
Cloud Types
  lxd
  maas
  manual
  openstack
  vsphere

Select cloud type: manual

Enter a name for your manual cloud: manual-cloud

Enter the ssh connection string for controller, username@<hostname or IP> or <hostname or IP>: test@192.168.89.7

Cloud "manual-cloud" successfully added to your local client.
```
verify clouds available:    

```
test@edge5:~$ juju clouds
Only clouds with registered credentials are shown.
There are more clouds, use --all to see them.
You can bootstrap a new controller using one of these clouds...

Clouds available on the client:
Cloud         Regions  Default    Type    Credentials  Source    Description
localhost     1        localhost  lxd     0            built-in  LXD Container Hypervisor
manual-cloud  1        default    manual  0            local 
```
Now bootstrap the `manual-cloud`:    

```
$ juju bootstrap manual-cloud
```
Add machines:    

```
test@edge5:~$ juju add-machine ssh:test@192.168.89.8
created machine 0
test@edge5:~$ juju add-machine ssh:test@192.168.89.9 && juju add-machine ssh:test@192.168.89.10
created machine 1
created machine 2
test@edge5:~$ juju machines
Machine  State    DNS            Inst id               Series  AZ  Message
0        started  192.168.89.8   manual:192.168.89.8   focal       Manually provisioned machine
1        started  192.168.89.9   manual:192.168.89.9   focal       Manually provisioned machine
2        started  192.168.89.10  manual:192.168.89.10  focal       Manually provisioned machine
```
Deploy microk8s via:    

```
$  juju deploy -n3 cs:~pjdc/microk8s
```
After deployment, the juju status show:    

```
test@edge5:~$ juju status
Model    Controller            Cloud/Region          Version  SLA          Timestamp
default  manual-cloud-default  manual-cloud/default  2.9.5    unsupported  17:27:04+08:00

App       Version  Status  Scale  Charm     Store       Channel  Rev  OS      Message
microk8s           active      3  microk8s  charmstore  stable    24  ubuntu  

Unit         Workload  Agent  Machine  Public address  Ports                     Message
microk8s/0*  active    idle   0        192.168.89.8    80/tcp,443/tcp,16443/tcp  
microk8s/1   active    idle   1        192.168.89.9    80/tcp,443/tcp,16443/tcp  
microk8s/2   active    idle   2        192.168.89.10   80/tcp,443/tcp,16443/tcp  

Machine  State    DNS            Inst id               Series  AZ  Message
0        started  192.168.89.8   manual:192.168.89.8   focal       Manually provisioned machine
1        started  192.168.89.9   manual:192.168.89.9   focal       Manually provisioned machine
2        started  192.168.89.10  manual:192.168.89.10  focal       Manually provisioned machine

```
Verify Ha:    

```
test@edge5:~$ juju exec --application microk8s -- 'microk8s status | grep -A2 high-availability:'
- return-code: 0
  stdout: |
    high-availability: yes
      datastore master nodes: 192.168.89.8:19001 192.168.89.9:19001 192.168.89.10:19001
      datastore standby nodes: none
  unit: microk8s/0
- return-code: 0
  stdout: |
    high-availability: yes
      datastore master nodes: 192.168.89.8:19001 192.168.89.9:19001 192.168.89.10:19001
      datastore standby nodes: none
  unit: microk8s/1
- return-code: 0
  stdout: |
    high-availability: yes
      datastore master nodes: 192.168.89.8:19001 192.168.89.9:19001 192.168.89.10:19001
      datastore standby nodes: none
  unit: microk8s/2

```
Verify the k8s status:    

```
test@edge5:~$ juju exec --application microk8s -- microk8s kubectl get node
- return-code: 0
  stdout: |
    NAME    STATUS   ROLES    AGE   VERSION
    edge8   Ready    <none>   13m   v1.21.1-3+ba118484dd39df
    edge7   Ready    <none>   18m   v1.21.1-3+ba118484dd39df
    edge9   Ready    <none>   13m   v1.21.1-3+ba118484dd39df
  unit: microk8s/0
- return-code: 0
  stdout: |
    NAME    STATUS   ROLES    AGE   VERSION
    edge8   Ready    <none>   13m   v1.21.1-3+ba118484dd39df
    edge7   Ready    <none>   18m   v1.21.1-3+ba118484dd39df
    edge9   Ready    <none>   13m   v1.21.1-3+ba118484dd39df
  unit: microk8s/1
- return-code: 0
  stdout: |
    NAME    STATUS   ROLES    AGE   VERSION
    edge8   Ready    <none>   13m   v1.21.1-3+ba118484dd39df
    edge7   Ready    <none>   18m   v1.21.1-3+ba118484dd39df
    edge9   Ready    <none>   13m   v1.21.1-3+ba118484dd39df
  unit: microk8s/2
```
