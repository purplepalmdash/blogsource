+++
title= "WorkingTipsOnKylinOptimized"
date = "2023-02-08T09:51:46+08:00"
description = "WorkingTipsOnKylinOptimized"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目的
关闭kylinv10桌面特效
### 方法
安装`qdbus`：    

```
$ sudo apt install -y qdbus-qt5
```
`/usr/bin`下新建脚本文件`switchoffeffect.sh`:     

```
#!/bin/bash
qdbus org.ukui.KWin /Compositor suspend
```

`/usr/share/applications`下复制一个desktop模板用于撰写自定义启动文件:    

```
$ sudo cp /usr/share/applications/kylin-weather.desktop /usr/share/applications/xxxxx-optimized.desktop
```
更改关键字段，最终效果如下:    

```
[Desktop Entry]
Name=SwichOff
Name[zh_CN]=XX优化
Name[bo_CN]=གནམ་གཤིས།
Comment[bo_CN]=ཀྲུང་གོའི་དུས་ལྟར་གནམ་གཤིས།
Comment=Indicator applet for current weather conditions in China
Comment[zh_CN]=关闭特效
GenericName[bo_CN]=གནམ་གཤིས།
GenericName=China Weather Applet
Keywords=weather,china
Exec=sh /usr/bin/switchoffeffect.sh
Icon=kylin-weather
Terminal=false
Type=Application
X-GNOME-Autostart-Delay=10
Categories=System;Utility;
X-UKUI-AutoRestart=true;
```
开始菜单中点击: `设置->应用->开机启动`, 点击`添加`，在弹出的`添加自定义程序`对话框中选择刚才创建的启动文件:    

![/images/2023_02_08_09_58_44_821x488.jpg](/images/2023_02_08_09_58_44_821x488.jpg)

点击确定后，开机启动项如下:    

![/images/2023_02_08_09_59_50_903x343.jpg](/images/2023_02_08_09_59_50_903x343.jpg)

重启后，桌面因为特效关闭的缘故，流畅度大有提升。
