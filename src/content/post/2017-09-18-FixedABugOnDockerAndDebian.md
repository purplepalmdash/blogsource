+++
title = "FixedABugOnDockerAndDebian"
date = "2017-09-18T09:36:59+08:00"
description = "FixedABugOnDockerAndDebian"
keywords = ["Linux"]
categories = ["Linux"]
+++
When install docker on Debian9, when configured the static ip address mode,
then you install docker, you will find the ip address won't be configured as
you imagined. It will get the dhcp ip from your dhcpd server. Following are
the steps for correct this bug:    

```
# systemctl disable docker.service
# systemctl enable rc-local.service
```
Then in `/etc/rc.local` file you will add following lines:    

```
#!/bin/sh -e
systemctl start docker.service
```
Then you should `chmod a+x /etc/rc.local`.    
Now restart machine, you will find your networking address is configured as
you defined in `/etc/network/interfaces`, and your docker start correctly.    
