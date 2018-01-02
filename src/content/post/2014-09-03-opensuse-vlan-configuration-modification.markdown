---
categories: ["opensuse"]
comments: true
date: 2014-09-03T00:00:00Z
title: OpenSuse vlan configuration modification
url: /2014/09/03/opensuse-vlan-configuration-modification/
---

The command is listed as following:     

```
linux:/etc/udev # cd rules.d/
linux:/etc/udev/rules.d # ls
10-network.rules  55-libsane.rules  56-sane-backends-autoconfig.rules  70-kpartx.rules  70-persistent-net.rules  99-iwlwifi-led.rules
linux:/etc/udev/rules.d # cat 70-persistent-net.rules
SUBSYSTEM=="net", ATTR{address}=="00:xx:xx:xx:xx", NAME="eth1", DRIVERS=="?*"
linux:/etc/udev/rules.d # cat /usr/lib/systemd/system/vlan.service
[Unit]
Description=Add Vlan Configure 
Type=idle

[Service]
ExecStart=/usr/bin/vconfig.sh

[Install]
WantedBy=multi-user.target

```
Change persistent name from enp0s25 to eth1 and let it possible to add vlan on this interface.    

And start the vlan configuration at the very last of the systemd working.   
