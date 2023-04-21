+++
title= "IDVdGPUReleaseNote"
date = "2023-04-21T09:35:15+08:00"
description = "IDVdGPUReleaseNote"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Create Cubic environment
Get the latest ubuntu desktop installation image:    

```
wget https://mirrors.ustc.edu.cn/ubuntu-releases/22.04.2/ubuntu-22.04.2-desktop-amd64.iso
mkdir ~/idv_dGPU_release
```
Choose the newly created directory:    

![/images/2023_04_21_09_36_41_582x193.jpg](/images/2023_04_21_09_36_41_582x193.jpg)

Fill in descriptions:    

![/images/2023_04_21_09_37_35_932x603.jpg](/images/2023_04_21_09_37_35_932x603.jpg)

Now we get the virtual environment terminal:    

![/images/2023_04_21_09_38_26_934x196.jpg](/images/2023_04_21_09_38_26_934x196.jpg)

### Customization
Change repository and update to latest:    

```
root@cubic:~# cat /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/ubuntu jammy main restricted
deb http://mirrors.ustc.edu.cn/ubuntu jammy-updates main restricted
deb http://mirrors.ustc.edu.cn/ubuntu jammy universe
deb http://mirrors.ustc.edu.cn/ubuntu jammy-updates universe
deb http://mirrors.ustc.edu.cn/ubuntu jammy multiverse
deb http://mirrors.ustc.edu.cn/ubuntu jammy-updates multiverse
deb http://mirrors.ustc.edu.cn/ubuntu jammy-backports main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu jammy-security main restricted
deb http://mirrors.ustc.edu.cn/ubuntu jammy-security universe
deb http://mirrors.ustc.edu.cn/ubuntu jammy-security multiverse
root@cubic:~# apt install -y kubuntu-desktop virt-manager build-essential
```
Change default to sddm:    

![/images/2023_04_21_09_49_57_942x475.jpg](/images/2023_04_21_09_49_57_942x475.jpg)



### Verification

![/images/2023_04_21_10_18_31_635x215.jpg](/images/2023_04_21_10_18_31_635x215.jpg)

Display:   

![/images/2023_04_21_10_19_15_1286x433.jpg](/images/2023_04_21_10_19_15_1286x433.jpg)

VT-d && VT-x:    

![/images/2023_04_21_10_20_17_1345x495.jpg](/images/2023_04_21_10_20_17_1345x495.jpg)

Intel SGX:    

![/images/2023_04_21_10_20_42_1196x297.jpg](/images/2023_04_21_10_20_42_1196x297.jpg)

Install(Install Ubuntu):     

![/images/2023_04_21_10_46_06_890x763.jpg](/images/2023_04_21_10_46_06_890x763.jpg)

Normal install:    

![/images/2023_04_21_10_46_33_1364x744.jpg](/images/2023_04_21_10_46_33_1364x744.jpg)

Use nvme ssd:    

![/images/2023_04_21_10_47_52_1140x596.jpg](/images/2023_04_21_10_47_52_1140x596.jpg)

Try模式进入后，停用:    

![/images/2023_04_21_11_00_21_1126x700.jpg](/images/2023_04_21_11_00_21_1126x700.jpg)

删除:    

![/images/2023_04_21_11_00_39_1112x752.jpg](/images/2023_04_21_11_00_39_1112x752.jpg)

应用:    

![/images/2023_04_21_11_00_54_927x598.jpg](/images/2023_04_21_11_00_54_927x598.jpg)

安装:    

![/images/2023_04_21_11_01_12_439x289.jpg](/images/2023_04_21_11_01_12_439x289.jpg)

现在可以选择并继续:     

![/images/2023_04_21_11_01_53_915x414.jpg](/images/2023_04_21_11_01_53_915x414.jpg)

设置用户名/密码/自动登陆:    

![/images/2023_04_21_11_02_51_895x587.jpg](/images/2023_04_21_11_02_51_895x587.jpg)

等待安装完成后，拔掉U盘后重新启动。    

### 启动虚拟机
导文件:    

```
# scp ./mac_hdd_ng.img rx560.qcow2 rx550_OVMF_VARS-1024x768.fd OVMF_CODE.fd test@192.168.1.123:~
# virsh dumpxml macOSrx550>aaaa.xml
# scp ./aaaa.xml test@192.168.1.123:~

```
