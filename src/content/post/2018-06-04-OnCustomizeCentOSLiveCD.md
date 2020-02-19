+++
title = "OnCustomizeCentOSLiveCD"
date = "2018-06-04T09:16:28+08:00"
description = "OnCustomizeCentOSLiveCD"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Preparation
A. Create local repository for centos installation:   

Install `createrepo_c`on archlinux:    

```
# yaout createrepo_c
```
Create the repo for local installation:    

```
# sudo mount -t iso9660 -o loop CentOS-7-x86_64-Everything-1804.iso /mnt2
# cd /mnt2
# find . | grep rpm$ | xargs -I % cp % /var/download/centos1804rpms
# cd /var/download/centos1804rpms
# createrepo_c .
```
Use this repo:    

```
# mkdir /etc/yum.repos.d/back && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/back/
# vim /etc/yum.repos.d/base.repo 
[base]
name=base
baseurl=http://192.168.122.1/centos1804rpms
enabled=1
gpgcheck=0
```

B. Prepare kickstart repository

```
# git clone https://github.com/CentOS/sig-core-livemedia.git
# cd sig-core-livemedia
# cd kickstarts
# wget https://gist.githubusercontent.com/lunatilia/8a195ccb0b415cbbd94f/raw/e8ede17f331dc8fe43012e7e4f8123a02e6bedc8/centos-7-livedvd-jp.cfg
```

C. Edit the cfg file, the modified file is listed in following position:    

### Make system
In chroot terminal, do following:    

```
#  yum remove -y libreoffice-gtk3-5.3.6.1-10.el7.x86_64 libreoffice-draw-5.3.6.1-10.el7.x86_64 libreoffice-ure-5.3.6.1-10.el7.x86_64 libreoffice-pyuno-5.3.6.1-10.el7.x86_64 libreoffice-xsltfilter-5.3.6.1-10.el7.x86_64 libreoffice-gtk2-5.3.6.1-10.el7.x86_64 libreoffice-graphicfilter-5.3.6.1-10.el7.x86_64 libreoffice-math-5.3.6.1-10.el7.x86_64 libreoffice-filters-5.3.6.1-10.el7.x86_64 libreoffice-data-5.3.6.1-10.el7.noarch libreoffice-core-5.3.6.1-10.el7.x86_64 libreoffice-pdfimport-5.3.6.1-10.el7.x86_64 libreoffice-calc-5.3.6.1-10.el7.x86_64 libreofficekit-5.3.6.1-10.el7.x86_64 libreoffice-ure-common-5.3.6.1-10.el7.noarch libreoffice-langpack-en-5.3.6.1-10.el7.x86_64 libreoffice-impress-5.3.6.1-10.el7.x86_64 libreoffice-opensymbol-fonts-5.3.6.1-10.el7.noarch libreoffice-x11-5.3.6.1-10.el7.x86_64 libreoffice-writer-5.3.6.1-10.el7.x86_64
# yum remove -y cheese
# cd /usr/local/
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/compose.tar.gz .
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/kismaticpkgs.tar.gz .
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/images.tar.gz . 
# cd /var/lib
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/portus.tar.gz .
# tar xzvf portus.tar.gz 
# rm -f portus.tar.gz 
# cd /usr/local/
# tar xzvf compose.tar.gz 
# tar xzvf images.tar.gz 
# tar xzvf kismaticpkgs.tar.gz 
# rm -f *.tar.gz
# cd /root/
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/pipcache.tar.gz .
# tar xzvf pipcache.tar.gz 
# pip install --no-index --find-links /root/pipcache docker-compose
# rm -f pipcache.tar.gz 
# rm -rf pipcache/
# cd /bin/
# scp root@192.192.189.128:/media/sdb/docker/loaddocker.sh .
# chmod 777 loaddocker.sh 
# cat loaddocker.sh 
    if [[ $(sudo docker images | grep registry) ]]; then
        echo "there are files"
    else
        docker load</usr/local/images/nginx.tar.bz2
        docker load</usr/local/images/1.tar
        docker load</usr/local/images/2.tar
        docker load</usr/local/images/3.tar
        docker load</usr/local/images/4.tar
        docker run --name docker-nginx -p 8888:80 -d -v /usr/local/kismaticpkgs:/usr/share/nginx/html jrelva/nginx-autoindex
        sed -i s/10.168.100.145/`hostname -I|awk '{print $1}'`/g /usr/local/compose/docker-compose.yml
    fi

# cd /etc/systemd/system/
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/mynginx.service .
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/docker-infra.service .
# systemctl enable sshd.service
# cd /root
# scp root@192.192.189.128:/media/sdd/kvm_storage/ansible_kismatic/ansible_kismatic1110/kismatic.tar.gz .
# yum install -y ntp
# vim /etc/ntp.conf
# systemctl enable ntpd
# systemctl disable chronyd
# systemctl disable firewalld
# vi /etc/selinux/config
     disable the selinux configuration
```
Exit the chroot, the program will continue to build the iso.   

### boot-up the system
After bootup the system, install the os via clicking the live install icon on the desktop.   

In newly installed system, do following things:    

```
# systemctl enable docker
# systemctl start docker
# /bin/loaddocker.sh
# systemctl enable mynginx.service
# systemctl enable docker-infra.service
# reboot
```
Now you could check your installation.    


Sometimes the boot will fail, I don't know why, maybe because of the dracut issue?    
