+++
title= "BuildingBlissOS"
date = "2022-07-26T14:22:55+08:00"
description = "BuildingBlissOS"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
Install necessray packages:    
```
sudo apt install -y openjdk-8-jdk build-essential
sudo apt install git-core gnupg flex bison maven gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip squashfs-tools  libssl-dev ninja-build lunzip syslinux syslinux-utils gettext genisoimage gettext bc xorriso libncurses5 xmlstarlet build-essential git imagemagick lib32readline-dev lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev libxml2 lzop pngcrush rsync schedtool  python3-mako libelf-dev$a
sudo apt-get install gcc-10 g++-10
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
Get the repo from `ustc.edu.cn`, and replace the repo for proxy.  

```
$ curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
$ chmod a+x ~/bin/repo
## 如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：
## REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'
``` 

Repo init:    

```
repo init -u https://github.com/BlissRoms-x86/manifest.git -b r11-r36
```
Repo sync(Using redsocks):    

```
repo sync -j8
```
After repo sync:    

```
cd prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.6
git checkout 2078a6bf9e5479104cfe2cbf54e9602672bd89f7
```
Build via:    

```
. build/envsetup.sh
lunch android_x86_64-userdebug
export NO_KERNEL_CROSS_COMPILE=true
export BLISS_BUILD_VARIANT=foss
mka iso_img -j8
```
The generated iso is listed as:    

```
out/target/product/x86_64/BlissOS-14.3-x86_64-202207270925_k-_m-r11-r36.iso is built successfully.
#### build completed successfully (02:40:00 (hh:mm:ss)) ####

dash@2204:/media/sda/Code/BlissOS$ 
dash@2204:/media/sda/Code/BlissOS$ ls out/target/product/x86_64/BlissOS-14.3-x86_64-202207270925_k-_m-r11-r36.iso
out/target/product/x86_64/BlissOS-14.3-x86_64-202207270925_k-_m-r11-r36.iso
```
