+++
title= "WorkingTipsOnGuestVM"
date = "2022-05-23T10:01:03+08:00"
description = "WorkingTipsOnGuestVM"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Add Channel
Add a channel named `Channel spice`:    

![/images/2022_05_23_10_02_46_551x307.jpg](/images/2022_05_23_10_02_46_551x307.jpg)

Install python and let python2 to be the default python version:     

```
# sudo apt-get install -y python2
# update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
update-alternatives: using /usr/bin/python2.7 to provide /usr/bin/python (python) in auto mode
# python
Python 2.7.18 (default, Mar 12 2022, 06:24:29) 
[GCC 11.2.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> quit()
# sudo apt-get install -y libevent-dev
# cd /usr/ && cp ./lib/aarch64-linux-gnu/libevent-2.1.so.7 ./lib/aarch64-linux-gnu/libevent-2.1.so.6
```
