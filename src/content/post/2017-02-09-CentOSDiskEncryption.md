+++
keywords = ["Encryption"]
description = "Disk Encryption Of Ubuntu"
date = "2017-02-09T14:37:24+08:00"
title = "CentOS全盘加密"
categories = ["Linux"]

+++
以下记录安装好加密磁盘后，免密码登录配置。    

### keyfile生成及配置
生成加密keyfile:    

```
# dd if=/dev/urandom of=/boot/keyfile bs=1024 count=4
```
用加密的keyfile解锁加密盘:    

```
# cryptsetup luksAddKey /dev/vda2 /boot/keyfile --key-slot 1
Enter any existing passphrase: 
```
### dracut配置
更改`/usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh`文件:    

```
info "luksOpen $device $luksname $luksfile $luksoptions"

+++ # Unlock with USB key
+++ sleep 3
+++ udevsettle
+++ usbkey=/dev/disk/by-uuid/8cc8c3fe-8b6d-4adf-aab5-e3b9e758b622
+++ if [ -e $usbkey ]; then
+++   ask_passphrase=0
+++   echo "USB Key detected - unlocking partition $device ..."
+++   echo "mkdir"
+++   mkdir -p /mnt
+++   echo "mount"
+++   mount $usbkey /mnt
+++   echo "unlock"
+++   cat /mnt/keyfile | cryptsetup luksOpen $device $luksname --key-file=-
+++ fi
```

因为我们使用了几条命令，所以需要将可执行文件打包进去:    

```
# vi /usr/lib/dracut/modules.d/90crypt/module-setup.sh
    dracut_need_initqueue
+++    inst /usr/bin/cat
+++    inst /usr/bin/mkdir
+++    inst /usr/bin/mount
}
```

然后更改有关dracut的配置，  

```
# dracut modules to omit
omit_dracutmodules+="systemd"

# dracut modules to add to the default
add_dracutmodules+="crypt"
```
执行`dracut -f`生成新的initramfs，重新启动，机器将自动进入系统.    
