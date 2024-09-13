+++
title= "CuttlefishOnRPI"
date = "2024-09-11T09:30:53+08:00"
description = "CuttlefishOnRPI"
keywords = ["Technology"]
categories = ["Technology"]
+++
Change sshd configuration:    

```
$ sudo cat /etc/ssh/sshd_config.d/50-cloud-init.conf 
PasswordAuthentication yes
$ sudo sed -i 's@//ports.ubuntu.com@//mirrors.ustc.edu.cn@g' /etc/apt/sources.list.d/ubuntu.sources
```
`sudo apt update -y && sudo apt upgrade -y` , then reboot.   

Build cuttlefish:    

```
git clone https://github.com/google/android-cuttlefish
cd android-cuttlefish
tools/buildutils/build_packages.sh
```
Install:     

```
sudo dpkg -i ./cuttlefish-base_*_*64.deb || sudo apt-get install -f
sudo dpkg -i ./cuttlefish-user_*_*64.deb || sudo apt-get install -f
sudo usermod -aG kvm,cvdnetwork,render $USER
sudo apt install -y libvirglrenderer-dev libvirglrenderer1
sudo reboot
```
Prepare packages:    

```
mkdir cf
cd cf
tar xzvf /media/test/78739a87-9b0b-4935-902f-7d78cc09a076/home/test/cvd-host_package.tar.gz
unzip /media/test/78739a87-9b0b-4935-902f-7d78cc09a076/home/test/aosp_cf_arm64_only_phone-img-11489887.zip 
```
maybe should use `--gpu_mode=guest_swiftshader`
