+++
title = "UpgradeKernelForRHEL74"
date = "2020-02-10T09:41:32+08:00"
description = "UpgradeKernelForRHEL74"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Online Steps
rhel74, default kernel is:    

```
# uname -a
Linux node 3.10.0-693.el7.x86_64 #1 SMP Thu Jul 6 19:56:57 EDT 2017 x86_64 x86_64 x86_64 GNU/Linux
```
Configure repo and install newer kernel:    

```
# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# wget https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
# rpm -ivh elrepo-release-7.0-4.el7.elrepo.noarch.rpm
# yum update -y
# yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
Loaded plugins: product-id, search-disabled-repos, subscription-manager
This system is not registered with an entitlement server. You can use subscription-manager to register.
elrepo-kernel                                                                                                                          | 2.9 kB  00:00:00     
elrepo-kernel/primary_db                                                                                                               | 1.9 MB  00:00:58     
Available Packages
kernel-lt.x86_64                                                               4.4.213-1.el7.elrepo                                              elrepo-kernel
kernel-lt-devel.x86_64                                                         4.4.213-1.el7.elrepo                                              elrepo-kernel
kernel-lt-doc.noarch                                                           4.4.213-1.el7.elrepo                                              elrepo-kernel
kernel-lt-headers.x86_64                                                       4.4.213-1.el7.elrepo                                              elrepo-kernel
kernel-lt-tools.x86_64                                                         4.4.213-1.el7.elrepo                                              elrepo-kernel
kernel-lt-tools-libs.x86_64                                                    4.4.213-1.el7.elrepo                                              elrepo-kernel
kernel-lt-tools-libs-devel.x86_64                                              4.4.213-1.el7.elrepo                                              elrepo-kernel
kernel-ml-devel.x86_64                                                         5.5.2-1.el7.elrepo                                                elrepo-kernel
kernel-ml-doc.noarch                                                           5.5.2-1.el7.elrepo                                                elrepo-kernel
kernel-ml-headers.x86_64                                                       5.5.2-1.el7.elrepo                                                elrepo-kernel
kernel-ml-tools.x86_64                                                         5.5.2-1.el7.elrepo                                                elrepo-kernel
kernel-ml-tools-libs.x86_64                                                    5.5.2-1.el7.elrepo                                                elrepo-kernel
kernel-ml-tools-libs-devel.x86_64                                              5.5.2-1.el7.elrepo                                                elrepo-kernel
perf.x86_64                                                                    5.5.2-1.el7.elrepo                                                elrepo-kernel
python-perf.x86_64                                                             5.5.2-1.el7.elrepo                                                elrepo-kernel
# yum --enablerepo=elrepo-kernel install kernel-ml
# sudo sed -i 's/^GRUB_DEFAULT.*/GRUB_DEFAULT=0/' /etc/default/grub
# sudo grub2-mkconfig -o /boot/grub2/grub.cfg
# sudo reboot
```
Check the kernel version is `5.5.2-1`:     

```
# uname -a
Linux node 5.5.2-1.el7.elrepo.x86_64 #1 SMP Tue Feb 4 16:29:48 EST 2020 x86_64 x86_64 x86_64 GNU/Linux
# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.4 (Maipo)
```

### Offline Steps
Scp the rpm into the server and install it via:    

```
# scp ./kernel-ml-5.5.2-1.el7.elrepo.x86_64.rpm vagrant@xxx.xxx.xxx.xxx:/home/vagrant
# ssh into xxx.xxx.xxx.xxx
...................
$ sudo sed -i 's/^GRUB_DEFAULT.*/GRUB_DEFAULT=0/' /etc/default/grub
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.5.2-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.5.2-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-693.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-693.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-61b21bac36de423f82052de06e3a892b
Found initrd image:
/boot/initramfs-0-rescue-61b21bac36de423f82052de06e3a892b.img
done
$ sudo reboot
```
Check:    

```
$ uname -a
Linux node 5.5.2-1.el7.elrepo.x86_64 #1 SMP Tue Feb 4 16:29:48 EST 2020 x86_64 x86_64 x86_64 GNU/Linux
$ cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.4 (Maipo)
```

### Related packages
Manually download them from
`http://elrepo.reloumirrors.net/kernel/el7/x86_64/RPMS/`, related rpms is
listed as:    

```
# pwd
/media/sda/rhel74NewKernel
# ls
kernel-ml-5.5.2-1.el7.elrepo.x86_64.rpm
kernel-ml-tools-5.5.2-1.el7.elrepo.x86_64.rpm
kernel-ml-devel-5.5.2-1.el7.elrepo.x86_64.rpm
kernel-ml-tools-libs-devel-5.5.2-1.el7.elrepo.x86_64.rpm
```

### 中文更新步骤
主线内核可从
`http://elrepo.reloumirrors.net/kernel/el7/x86_64/RPMS/`下载，文件列表如下：    

```
# pwd
/media/sda/rhel74NewKernel
# ls
kernel-ml-5.5.2-1.el7.elrepo.x86_64.rpm
kernel-ml-headers-5.5.2-1.el7.elrepo.x86_64.rpm
kernel-ml-tools-libs-devel-5.5.2-1.el7.elrepo.x86_64.rpm
kernel-ml-devel-5.5.2-1.el7.elrepo.x86_64.rpm
kernel-ml-tools-5.5.2-1.el7.elrepo.x86_64.rpm
```
注： 如果只需要升级内核，则只需要 `kernel-ml-5.5.2-1.el7.elrepo.x86_64.rpm`
一个包就足够，如果编译时需要内核头文件依赖，则有可能需要其他几个包。可以根据需要自行安装。    

离线更新，上传包到rhel74服务器上，:    

```
# scp ./*.rpm vagrant@xxx.xxx.xxx.xxx:/home/vagrant
# ssh into xxx.xxx.xxx.xxx
...................
$ sudo yum install -y ./kernel-ml-5.5.2-1.el7.elrepo.x86_64.rpm
$ sudo sed -i 's/^GRUB_DEFAULT.*/GRUB_DEFAULT=0/' /etc/default/grub
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
$ sudo reboot
```
检查更新后结果:    

```
$ uname -a
Linux node 5.5.2-1.el7.elrepo.x86_64 #1 SMP Tue Feb 4 16:29:48 EST 2020 x86_64 x86_64 x86_64 GNU/Linux
$ cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.4 (Maipo)
```
