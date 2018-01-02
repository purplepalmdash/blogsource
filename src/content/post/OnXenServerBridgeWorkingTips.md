+++
categories = ["Virtualization"]
date = "2016-08-18T19:29:45+08:00"
description = "On Set XenServer Bridged virtual machines"
keywords = ["XenServer"]
title = "OnXenServerBridgeWorkingTips"

+++
### Background
I will use XenServer for testing, while I made a vagrant box of XenServer 6.5, it could
work properly in seperated networking, so following I will try to setup a bridged
"XenServer" which will acts like a real physical machine.     

### Vagrantfile
The configuration part is listed as:    

```
  # csagentxen65 node.
  # Add one networking, modify hostname, define memory, CPU cores.
  config.vm.define :csagentxen65 do |csagentxen65|
    csagentxen65.vm.box = "Xen65Box"
    csagentxen65.vm.boot_timeout = '36000'
    csagentxen65.vm.hostname = CLOUDSTACK_AGENT_HOSTNAME
    csagentxen65.vm.network :public_network, 
	    :dev => "br0",
	    :mode => "bridge",
	    :type => "bridge",
	    :ip => "192.168.10.3"
    # Disable mounting of vagrant folder as it's not supported on xenserver
    csagentxen65.vm.synced_folder ".", "/vagrant", disabled: true
    csagentxen65.vm.provider :libvirt do |domain|
      domain.memory = 8192
      domain.cpus = 4
      domain.storage_pool_name = 'XenStoragePool'
      domain.nested = true
      domain.cpu_mode = 'host-passthrough'
      domain.nic_model_type = 'e1000'
      domain.management_network_mac = "08:00:27:1D:90:A8"
    end
  end
```
Here we used our newly created storage pool, I use virt-manager for creating a new
storage pool, which is in a seprated disk, could gain much more IOPS.   

### Bug-Fix
vagrant-libvirt will create 2 ethernet port, while the default one is xenbr0, but we
want the xenbr1 be the management port.      

Gain a administration shell:    

![/images/2016_08_18_19_34_41_498x510.jpg](/images/2016_08_18_19_34_41_498x510.jpg)     
Enter following command:    

```
# ifconfig eth1 netmask 255.255.0.0
# route add default gw 192.168.0.xxx eth1
```

Now you can run ansible-playbook against the XenServer, but it will stuck.    

Now manually configure eth1's netmask and its gateway. ansible-playbook could be run
properly.    
