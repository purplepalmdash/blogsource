+++
title= "HostapdOnRTL8188EE"
date = "2023-08-31T09:26:58+08:00"
description = "HostapdOnRTL8188EE"
keywords = ["Technology"]
categories = ["Technology"]
+++
lspci for getting the wireless card mode:    

```
# lspci | grep -i wireless
01:00.0 Network controller: Realtek Semiconductor Co., Ltd. RTL8188EE Wireless Network Adapter (rev 01)
```
Install the script:    

```
yay -S linux-wifi-hotspot
```
Create the wifi via:     

```
systemctl enable --now create_ap
create_ap wlp1s0 enp4s0 xxx xxxxxxx --hidden
```
