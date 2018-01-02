+++
keywords = ["Encryption"]
date = "2017-02-10T18:08:48+08:00"
description = "Working tips on encryption"
title = "WorkingTipsOnEncryption"
categories = ["Linux"]

+++
Refers to:    

[https://blog.tinned-software.net/automount-a-luks-encrypted-volume-on-system-start/](https://blog.tinned-software.net/automount-a-luks-encrypted-volume-on-system-start/)    

### Disk Partition Encryption
Steps for encryption of vdb1:    

```
# dd if=/dev/urandom of=/root/vdb_secret_key bs=512 count=8
# cryptsetup -v luksAddKey /dev/vdb1 /root/vdb_secret_key
# cryptsetup luksDump /dev/vdb1 | grep "Key Slot"
# cryptsetup -v luksOpen /dev/vdb1 vdb1_crypt --key-file=/root/vdb_secret_key 
# cryptsetup -v luksClose vdb1_crypt
```
Add following line for auto decryption:    

```
# vim /etc/crypttab
vdb1_crypt UUID=43740d4f-df91-492e-8d06-b32f461a633e /root/vdb_secret_key luks
```
While UUID is generated via following command:    

```
# cryptsetup luksDump /dev/vdb1  | grep "UUID"
```
Add lines into `/etc/fstab`:    

```
/dev/mapper/vdb1_crypt	/media/vdb1	ext4	defaults	0	 2
```

### Volume Encryption
For storing contents in an encrypted file, do following steps:    

```
# dd if=/dev/zero of=/root/luks.vol bs=1M count=1024
# cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 luksFormat /root/luks.vol
# cryptsetup luksOpen /root/luks.vol file
# ls /dev/mapper/
# mkfs.ext4 /dev/mapper/file
```
Now we begin to use keyfile for unlock this partition:    

```
# dd if=/dev/urandom of=/root/file_key bs=512 count=8
# cryptsetup -v luksAddKey /root/luks.vol /root/file_key
# cryptsetup -v luksOpen /root/luks.vol vol_crypt --key-file=/root/file_key 
# cryptsetup -v luksClose vol_crypt
```
Get the UUID of the luks.vol:    

```
# cryptsetup luksDump /root/luks.vol  | grep "UUID"
```
Now you could add following lines into `/etc/rc.local`:    

```
cryptsetup -v luksOpen /root/luks.vol vol_crypt --key-file=/root/file_key
mount /dev/mapper/vol_crypt /media/vol
```
