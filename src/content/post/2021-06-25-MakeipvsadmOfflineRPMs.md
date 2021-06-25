+++
title= "MakeipvsadmOfflineRPMs"
date = "2021-06-25T16:16:56+08:00"
description = "MakeipvsadmOfflineRPMs"
keywords = ["Technology"]
categories = ["Technology"]
+++
现场测试时，测试人员报告在安装ipvsadm时，有包缺失现象。报错现象如下:    

![/images/2021_06_26_06_58_40_903x356.jpg](/images/2021_06_26_06_58_40_903x356.jpg)

原因是因为在安装ipvsadm时的libnl3依赖更新开始，客户安装的操作系统是centos7.2,而我们做包的系统是>centos7.5以后的，因而导致了libnl3-cli因libnl3的更新抱怨更新后确实依赖而不能进行安装。

解决方案：   

![/images/2021_06_26_06_57_10_1704x154.jpg](/images/2021_06_26_06_57_10_1704x154.jpg)

