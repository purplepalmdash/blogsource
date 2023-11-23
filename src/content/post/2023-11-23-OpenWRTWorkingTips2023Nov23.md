+++
title= "OpenWRTWorkingTips2023Nov23"
date = "2023-11-23T10:24:38+08:00"
description = "OpenWRTWorkingTips2023Nov23"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. release space
disk is almost full:    

```
root@OpenWrt:~# df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/root                 2.5M      2.5M         0 100% /rom
tmpfs                    60.8M     92.0K     60.8M   0% /tmp
/dev/mtdblock3            3.9M      3.5M    396.0K  90% /overlay
overlayfs:/overlay        3.9M      3.5M    396.0K  90% /
```
Use following command for getting the size of installed app:     

```
# cat /tmp/list.sh
#!/bin/sh
 
packagelist=$(ls /usr/lib/opkg/info/*.list)
 
for packagelistpath in $packagelist
do
        packagefiles=$(cat $packagelistpath)
        packagename=$(echo $packagelistpath | sed 's/\/usr\/lib\/opkg\/info\///g' | sed 's/\.list//g')
        if [[ -z "$packagefiles" ]]
        then
                packagesizeb=$(opkg info $packagename | grep Size\: | sed 's/Size\:\ //g')
                if [[ -z $packagesizeb ]]
                then
                        echo -e "-.- KB \t\t$packagename"
                else
                        packagesizekb=$(opkg info $packagename | grep Size\: | sed 's/Size\:\ //g' | awk 'END {printf "%.02f\n", $1/1024}')
                        printf '%-16s' "$packagesizekb KB"; printf '%s\n' "$packagename"
                fi
        else
                packagesizekb=$(for file in $packagefiles; do if [[ -f $file ]]; then du -k $file | cut -f1; fi; done | awk '{s+=$1} END {printf "%.02f\n", s}')
                printf '%-16s' "$packagesizekb KB"; printf '%s\n' "$packagename"
        fi
done | sort -n
# /tmp/list.sh
.....

551.00 KB       kmod-fs-ext4
# opkg remove kmod-fs-ext4
Removing package kmod-fs-ext4 from root...
# df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/root                 2.5M      2.5M         0 100% /rom
tmpfs                    60.8M     92.0K     60.8M   0% /tmp
/dev/mtdblock3            3.9M      3.2M    684.0K  83% /overlay
overlayfs:/overlay        3.9M      3.2M    684.0K  83% /
tmpfs                   512.0K         0    512.0K   0% /dev
.....
```
### 2. Install zerotier(Router)
Install zerotier:   

```
# opkg update
# opkg install zerotier
Installing zerotier (1.6.5-1) to root...
Downloading http://downloads.openwrt.org/releases/19.07.7/packages/mips_24kc/packages/zerotier_1.6.5-1_mips_24kc.ipk
......
# uci commit zerotier
```
Didn't configure the network transferring.     

### 3. Install zerotier(vps)
vps is debian, install via:    

```
curl -s https://install.zerotier.com | sudo bash
zerotier-cli  join xxxxxxxxx
``` 
### 4. Install zerotier(company computer)
ArchLinux, install via:   
 
```
sudo pacman -S zerotier-one
sudo systemctl start zerotier-one
sudo systemctl enable zerotier-one
sudo zerotier-cli join xxxxxx

```
### 5. Install zerotier(company workstation)
Ubuntu 22.04, install via:    

```
curl -s https://install.zerotier.com | sudo bash
sudo zerotier-cli join xxxxxx
```
Then change the config.json's ip address from ipv6 to zero-tier's ip address.   
