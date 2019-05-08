+++
title = "MinimumRHEL74DVD"
date = "2019-05-08T09:09:56+08:00"
description = "MinimumRHEL74DVD"
keywords = ["Linux"]
categories = ["Linux"]
+++
In a minimum installed system, do following:    

```
# mount /dev/sr0 /mnt
# vim  /etc/yum.repos.d/local.repo 
[local]
name=local
baseurl=file:///mnt
enabled=1
gpgcheck=0
# yum makecache
# cat /mnt/repodata/764ce0e938d43b3e9cb1bcca13cf71630aac0c44149f4a61548f642df3c70858-comps-Server.x86_64.xml
```
Around Line 937-1031, get all of the packges:    

```
# cat list1.txt | awk -F '>' {'print $2'} | awk -F '<' {'print $1'} 
```
According to the output, install all of the packges:    

```
yum install -y Red_Hat_Enterprise_Linux-Release_Notes-7-en-US
yum install -y audit
.......
yum install -y tboot
yum install -y NetworkManager-config-server
```

Now do following for getting all of the packages:    

```
# for i in `rpm -qa`; do echo $i.rpm|xargs -I % cp /mnt/Packages/% /root/pkgs; done
```
Added  `Base` group:    

```
# sed -n 483,632p /mnt/repodata/764ce0e938d43b3e9cb1bcca13cf71630aac0c44149f4a61548f642df3c70858-comps-Server.x86_64.xml  | awk -F '>' {'print $2'} | awk -F '<' {'print $1'} | xargs -I % yum install -y %
# for i in `rpm -qa`; do echo $i.rpm|xargs -I % cp /mnt/Packages/% /root/pkgs; done
```

Upload the Packages to building server, then:    

```
# mv repodata/ /root/whole_repodata
# mv Packages/ /root/whole_Packages
# mv /root/thinrepodata/ repodata
# mv /root/pkgs/ Packages
# cp repodata/764ce0e938d43b3e9cb1bcca13cf71630aac0c44149f4a61548f642df3c70858-comps-Server.x86_64.xml .
# createrepo -g 764ce0e938d43b3e9cb1bcca13cf71630aac0c44149f4a61548f642df3c70858-comps-Server.x86_64.xml  .
# mkisofs -o /root/new.iso  -b isolinux/isolinux.bin -c isolinux/boot.cat --no-emul-boot --boot-load-size 4 --boot-info-table -J -R -V "RHEL-7.4\x20Server.x86_64" . 
```

Your /root/new.iso will be much more smaller, cause it only contains `Core`
and `Base` group packages.    
