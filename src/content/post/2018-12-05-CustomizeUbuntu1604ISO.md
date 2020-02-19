+++
title = "Ubuntu1604ISOCustomize"
date = "2018-12-05T09:13:56+08:00"
description = "Ubuntu1604ISOCustomize"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Steps
ArchLinux Preparation:    

```
$ yaourt mkpasswd
```


```
# mkdir aiiso
# cd aiiso
# mv ../ubuntu-16.04.5-server-amd64.iso .
# mkdir newISO
# mkdir iso
# mount -o loop ./ubuntu-16.04.5-server-amd64.iso ./iso
# cp -r ./iso/* ./newISO
# cp -r ./iso/.disk ./newISO
# umount ./iso
# cp xxx.seed ./newISO/preseed
###### Make some customization
#  md5sum ./newISO/preseed/xxx.seed
ed9a5e91f66451080d27d3d85032801d  xxx.seed
# vim preseed/xxx.seed
# vim isolinux/txt.cfg
# mkisofs -D -r -V "NETSON_UBUNTU" -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o ../ai_ubuntu.iso .  > /dev/null 2>&1
# isohybrid ../ai_ubuntu.iso

```
