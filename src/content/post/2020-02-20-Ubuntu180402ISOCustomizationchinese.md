+++
title = "定制化Ubuntu 18.04.2 iso"
date = "2020-02-20T21:52:53+08:00"
description = "CustomizeUbuntuISO"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 准备
Ubuntu18.04.2 amd64 安装 iso, md5sum:    

```
34416ff83179728d54583bf3f18d42d2  ubuntu-18.04.2-server-amd64.iso
```
poweriso linux 版本:       

[http://www.poweriso.com/download-poweriso-for-linux.htm](http://www.poweriso.com/download-poweriso-for-linux.htm)
### 创建步骤
使用 poweriso 打开原始ISO：  

![/images/2020_02_20_21_54_17_797x300.jpg](/images/2020_02_20_21_54_17_797x300.jpg)

preseed 目录修改:    

![/images/2020_02_20_21_54_42_581x201.jpg](/images/2020_02_20_21_54_42_581x201.jpg)

添加txt.cfg:    

![/images/2020_02_20_21_58_54_596x431.jpg](/images/2020_02_20_21_58_54_596x431.jpg)

boot/grub配置文件更改，uefi模式需要:    

![/images/2020_02_20_22_00_30_617x448.jpg](/images/2020_02_20_22_00_30_617x448.jpg)

编辑完毕后，另存为新文件名(`xxxx-18.04.2-server-amd64-uefi.iso )    

### 验证(virtualbox)
虚拟机配置为uefi启动:    

![/images/2020_02_20_22_19_06_718x484.jpg](/images/2020_02_20_22_19_06_718x484.jpg)

选择含 `sep-small`的选项, `/` 大小 100G, `/var` 占据剩余空间:    

![/images/2020_02_20_22_19_59_640x276.jpg](/images/2020_02_20_22_19_59_640x276.jpg)

安装自动进行，完毕后验证用户名/密码，磁盘空间：    

![/images/2020_02_20_22_26_16_600x277.jpg](/images/2020_02_20_22_26_16_600x277.jpg)



