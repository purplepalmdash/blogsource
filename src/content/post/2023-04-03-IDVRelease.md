+++
title= "IDVRelease"
date = "2023-04-03T09:40:50+08:00"
description = "IDVRelease"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install cubic(ubuntu 22.04.2):    

```
sudo apt-add-repository universe
sudo apt-add-repository ppa:cubic-wizard/release
sudo apt update
sudo apt install --no-install-recommends cubic
```
Then start cubic and following the guideline.    

![/images/2023_04_03_09_41_47_744x543.jpg](/images/2023_04_03_09_41_47_744x543.jpg)

![/images/2023_04_03_09_42_46_960x651.jpg](/images/2023_04_03_09_42_46_960x651.jpg)

In terminal do:    

```
root@cubic:~# history
    1  dpkg -l
    2  gsettings set org.gnome.desktop.session idle-delay 0
    3  gsettings set org.gnome.desktop.screensaver ubuntu-lock-on-suspend false
    4  vi /etc/default/apport 
    5  passwd
    6  which vim
    7  apt install openssh-server
    8  vim /etc/ssh/sshd_config 
    9  vi /etc/ssh/sshd_config 
   10  dpkg -l | grep thunderbird
   11  sudo apt remove thunderbird
   12  dpkg -l | grep libreoffice
   13  sudo apt remove libreoffice
   14  sudo apt remove libreoffice*
   15  sudo apt remove libreoffice-core
   16  dpkg -l | more
   17  apt install -y sddm
   18  apt-cache search sddm
   19  apt update
   20  vi /etc/apt/sources.list
   21  ap tupdate
   22  apt update
   23  apt install sddm
   24  apt-cache search sddm
   25  apt-cache search build-essential
   26  apt-get policy build-essential
   27  apt-get search build-essential
   28  apt-cache policy build-essential
   29  vi /etc/apt/sources.list
   30  apt-get update
   31  apt-cache search sddm
   32  sudo apt install -y sddm
   33  df -h
   34  history
   35  useradd -m ctyunidv
   36  passwd ctyunidv
   37  mkdir -p /etc/sddm.conf.d/
   38  vim /etc/sddm.conf.d/autologin.conf
   39  vi /etc/sddm.conf.d/autologin.conf
   40  apt install -y iotop
   41  history
   42  clear
   43  ls
   44  vi /etc/default/grub 
   45  update-grub2 
   46  vim /etc/initramfs-tools/modules 
   47  vi /etc/initramfs-tools/modules 
   48  update-initramfs -u -k all
   49  which scp
   50  scp ctyunidv@172.23.119.211:~/CtyunDesktopIDV_1.0.2_101000200_x64_03-17-11-20.deb .
   51  scp ctyunidv@172.23.119.211:~/ctgcd-clouddesktop-idvagent.war .
   52  ls
   53  sudo apt install -y ./CtyunDesktopIDV_1.0.2_101000200_x64_03-17-11-20.deb 
   54  ssh-keygen 
   55  cat /root/.ssh/id_rsa.pub 
   56  scp root@172.23.119.211:/root/ljr/linux-image-5.10.90-c1dc2c9a39ac_5.10.90-c1dc2c9a39ac-17_amd64.deb .
   57  apt install -y ./linux-image-5.10.90-c1dc2c9a39ac_5.10.90-c1dc2c9a39ac-17_amd64.deb 
   58  ls
   59  history

```
