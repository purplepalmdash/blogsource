+++
title = "TipsOnRongISO"
date = "2018-11-17T08:50:37+08:00"
description = "TipsOnRongISO"
keywords = ["Linux"]
categories = ["Technology"]
+++
### AIM
For building the kubespray offline all-in-one deploying iso.    

### Steps
1. Download 7.5.1804 ISO.    
2. Create a new virtual machine, kvm based, minimal installation, for getting the
minimal vm files.   
3. Install redsocks, compile it, then shutdown the minimal vm.    
4. Create a qcow2 file based on minimal vm, start the vm, then change the
   IP/Netmask/DNS.    
5. Start redsocks. Test the unlimited networking.    
6. Clone the kubespray repository, deploy the kubespray in all in one node.   
7. Fetch the rpms from the kubespray all-in-one node.   
8. Create a isolated networking and setup the offline environment.   
9. Modify the kubespray source code for offline deployment.   
10. Portus offline registry repo building. 
11. Static website for holding static files(rpms/hypekube).    
12. docker use intranet registry/ rpm use static website.   

### TobeDone
Steps for building deployment system.    

```
1. 
docker-compose
portus (docker-compose) composition files. 
portus images. 

2. 
inventory file(top layer)
kubespray files will be uploaded to deployment node. 

3. 
dns server setup
manually add dns server in all of the nodes. 

4. 
If initial environment is ok, then deploy environment will also be ok.  
```
