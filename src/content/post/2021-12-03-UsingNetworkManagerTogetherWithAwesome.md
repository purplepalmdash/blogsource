+++
title= "UsingNetworkManagerTogetherWithAwesome"
date = "2021-12-03T09:13:26+08:00"
description = "UsingNetworkManagerTogetherWithAwesome"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
用于在Ubuntu20.04上开启awesome, 并使用NetworkManager管理网络.    

安装必要的包:    

```
$ sudo apt-get install -y awesome sddm net-tools vim network-manager network-manager-gnome ifupdown
```
配置awesome:    

```
$ sudo cp /etc/xdg/awesome/rc.lua ~/.config/awesome/rc.lua
$ vim ~/.config/awesome/rc.lua
..........
modkey = "Mod4"

--- Just Run Once Programs.
function run_once(cmd)
  findme = cmd
  firstspace = cmd:find(" ")
  if firstspace then
    findme = cmd:sub(0, firstspace-1)
  end
  awful.util.spawn_with_shell("pgrep -u $USER -x " .. findme .. " > /dev/null || (" .. cmd .. ")")
end

run_once("nm-applet &")
.............
```
配置NetworkManager:    

```
$ sudo vim /etc/NetworkManager/NetworkManager.conf
..........
[ifupdown]
managed=true
.........
```
更改netplan renderer方式:    

```
$ sudo vim /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  version: 2
  renderer: NetworkManager
```

现在重启物理机后， 手工配置NetworkManager:   

![/images/2021_12_03_09_18_42_276x236.jpg](/images/2021_12_03_09_18_42_276x236.jpg)

![/images/2021_12_03_09_19_03_606x170.jpg](/images/2021_12_03_09_19_03_606x170.jpg)

![/images/2021_12_03_09_19_29_771x342.jpg](/images/2021_12_03_09_19_29_771x342.jpg)

检查可看到配置生效。
