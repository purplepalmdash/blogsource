+++
categories = ["Linux"]
date = "2016-09-05T10:42:24+08:00"
description = "Playing RHEL"
keywords = ["Linux"]
title = "RHEL Working Tips"

+++
### Subscrition Manager
After install RHEL7, we could register it first:    

```
# subscription-manager register
```
List and register with the available pools:    

```
# subscription-manager list --available --all
# subscription-manager subscribe --pool=XXXXXXXXX
```
After registration, we could see all of the registered repos:    

```
$ subscription-manager repos --list
```

### Syncing Repos
Because the openstack repo is the paid channel, skip for other options.    
