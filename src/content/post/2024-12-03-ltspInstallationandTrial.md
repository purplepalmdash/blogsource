+++
title= "ltspInstallationandTrial"
date = "2024-12-03T11:20:25+08:00"
description = "ltspInstallationandTrial"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 安装ltspserver
Configuration:    

![/images/20241203_112055_x.jpg](/images/20241203_112055_x.jpg)

Installation:   

![/images/20241203_112039_x.jpg](/images/20241203_112039_x.jpg)

选择正常安装:    

![/images/20241203_112128_x.jpg](/images/20241203_112128_x.jpg)

用户名创建:    

![/images/20241203_112203_x.jpg](/images/20241203_112203_x.jpg)

配置:    

```
sudo apt update
sudo apt install -y openssh-server vim nethogs iotop
sudo add-apt-repository ppa:ltsp && sudo apt update
sudo apt install --install-recommends ltsp ltsp-binaries dnsmasq nfs-kernel-server openssh-server squashfs-tools ethtool net-tools epoptes
sudo usermod -aG epoptes test
sudo apt upgrade -y && sudo shutdown -h now
```
添加一个isolated网卡:    

![/images/20241203_113522_x.jpg](/images/20241203_113522_x.jpg)

在网络管理下，配置其ip地址为10.17.18.18：    

![/images/20241203_113929_x.jpg](/images/20241203_113929_x.jpg)


```
sudo ltsp dnsmasq --proxy-dhcp=0
sudo vim /etc/dnsmasq.d/ltsp-dnsmasq.conf
    dhcp-range=10.17.18.20,10.17.18.250,12h
sudo ltsp image /
sudo ltsp ipxe
sudo ltsp nfs
sudo ltsp initrd
sudo useradd -m test1
sudo useradd -m test2
sudo ltsp initrd
```

### 测试

![/images/20241203_114432_x.jpg](/images/20241203_114432_x.jpg)

4C 4G：    

![/images/20241203_114456_x.jpg](/images/20241203_114456_x.jpg)

命名:    

![/images/20241203_114516_x.jpg](/images/20241203_114516_x.jpg)
网络:   

![/images/20241203_114552_x.jpg](/images/20241203_114552_x.jpg)

引导:    

![/images/20241203_114613_x.jpg](/images/20241203_114613_x.jpg)

### virtualbox创建镜像

![/images/20241203_142035_x.jpg](/images/20241203_142035_x.jpg)

![/images/20241203_142130_x.jpg](/images/20241203_142130_x.jpg)
 

![/images/20241203_142428_x.jpg](/images/20241203_142428_x.jpg)

![/images/20241203_142443_x.jpg](/images/20241203_142443_x.jpg)

![/images/20241203_142550_x.jpg](/images/20241203_142550_x.jpg)

pre-allocate Full size:   

![/images/20241203_142609_x.jpg](/images/20241203_142609_x.jpg)

10G size:    

![/images/20241203_142629_x.jpg](/images/20241203_142629_x.jpg)

![/images/20241203_142908_x.jpg](/images/20241203_142908_x.jpg)

enable 3d acceleration.    

虚机分区:    

![/images/20241203_143521_x.jpg](/images/20241203_143521_x.jpg)

其他按默认安装完毕。   

gnome桌面:   

![/images/20241203_143731_x.jpg](/images/20241203_143731_x.jpg)

进入系统后，关闭lock/sleep等选项。    

```
cd /srv/ltsp/
ls
rm -f debian12_1.img 
ln -s /home/test/ubuntu1-flat.vmdk ./ubuntu1.img
ltsp image ubuntu1
ltsp ipxe
```
Next time reboot then you could enter ubuntu1.   

debian got some errors.    

kylin problem:     

```
root@ltscserver:/srv/ltsp# ln -s /home/test/kylin-flat.vmdk ./kylin.img
root@ltscserver:/srv/ltsp# ltsp image kylin
Running: losetup -rP /dev/loop6 /srv/ltsp/kylin.img
Running: mount -t tmpfs -o mode=0755 tmpfs /tmp/tmp.sV6xy2UAD1/tmpfs
Running: mount -t ext4 -o ro,noload /dev/loop6p1 /tmp/tmp.sV6xy2UAD1/tmpfs/0/looproot
Running: mount -t overlay -o upperdir=/tmp/tmp.sV6xy2UAD1/tmpfs/0/up,lowerdir=/tmp/tmp.sV6xy2UAD1/tmpfs/0/looproot,workdir=/tmp/tmp.sV6xy2UAD1/tmpfs/0/work /tmp/tmp.sV6xy2UAD1/tmpfs /tmp/tmp.sV6xy2UAD1/root/
Cleaning up kylin before mksquashfs...
Traceback (most recent call last):
  File "/usr/share/ltsp/client/login/pwmerge", line 440, in <module>
    main(sys.argv)
  File "/usr/share/ltsp/client/login/pwmerge", line 434, in main
    pwm = PwMerge(args[0], args[1], args[2], **dopts)
  File "/usr/share/ltsp/client/login/pwmerge", line 117, in __init__
    self.dpasswd, self.dgroup = self.read_dir(ddir, dur or dgr)
  File "/usr/share/ltsp/client/login/pwmerge", line 151, in read_dir
    with open("{}/passwd".format(xdir), "r") as file:
FileNotFoundError: [Errno 2] No such file or directory: '/tmp/tmp.sV6xy2UAD1/root/etc/passwd'
LTSP command failed: /usr/share/ltsp/client/login/pwmerge --ltsp --quiet /tmp/tmp.sV6xy2UAD1/root/tmp/pwempty /tmp/tmp.sV6xy2UAD1/root/etc /tmp/tmp.sV6xy2UAD1/root/tmp/pwmerged
Aborting ltsp

``` 

### kylin tips
Add repository:     

```
wget https://ltsp.org/misc/ltsp-ubuntu-ppa-focal.list -O /etc/apt/sources.list.d/ltsp-ubuntu-ppa-focal.list
wget https://ltsp.org/misc/ltsp_ubuntu_ppa.gpg -O /etc/apt/trusted.gpg.d/ltsp_ubuntu_ppa.gpg
apt update
sudo apt install --install-recommends ltsp ltsp-binaries dnsmasq nfs-kernel-server openssh-server squashfs-tools ethtool net-tools epoptes
sudo gpasswd -a test epoptes
```
