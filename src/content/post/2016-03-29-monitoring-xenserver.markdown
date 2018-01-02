---
categories: ["Virtualization"]
comments: true
date: 2016-03-29T12:23:50Z
title: Monitoring XenServer
url: /2016/03/29/monitoring-xenserver/
---

### Prerequisition
Refers
to[https://github.com/crashdump/collectd-xenserver](https://github.com/crashdump/collectd-xenserver)     

* Collected 4.9 or later
* Python2.4 Or Later
* sudo pip install XenAPI
* sudo pip install collectd

### Configuration
Make a directory under `/etc/collectd` folder, and copy the
`collectd-xenserver.py` into this folder:    

```
$ sudo mkdir -p  /var/collectd/plugins
$ sudo cp YourDictory/collectd-xenserver.py /var/collectd/plugins/collectd_xenserver.py
```
Now edit the configuration file of `/etc/collectd/collectd.conf`:    

```
<LoadPlugin python>
	Globals true
</LoadPlugin>

<Plugin python>
	ModulePath "/etc/collectd/plugins/"
	#LogTraces true
	#Interactive true
	Import "collectd_xenserver"
        <Module "collectd_xenserver">
              <Host "192.168.10.187">
                    User "root"
                    Password "xxxxx"
              </Host>
        </Module>

</Plugin>
```
Now restart the collectd, you will find the data-set has been collectd and
send into the graphite server.    

```
$ sudo service collectd restart
```
