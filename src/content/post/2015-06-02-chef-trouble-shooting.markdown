---
categories: ["Virtualization"]
comments: true
date: 2015-06-02T16:16:42Z
title: Chef Trouble-Shooting
url: /2015/06/02/chef-trouble-shooting/
---

### Error
Could not Add new nodes.     

### Reason
This is because the chefDK remains the old version of chef-client,     

```
[dash@~/chef-repo]$ chef --version
Chef Development Kit Version: 0.6.0
chef-client version: ERROR
berks version: ERROR
kitchen version: 1.4.0
```

### Solution
In node, manually get verified via following command:    

```
$ knife ssl fetch --config /etc/chef/client.rb
$ chef-client -l debug -S https://ChefServer/organizations/xxxxx -K /xxx/xxx/xxxxx.pem
```

Now bootstrap again, and you will see the node could be added into the Chef-Server's system.     
