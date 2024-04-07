+++
title= "x11dockerRunThunderbird"
date = "2024-04-07T09:45:49+08:00"
description = "x11dockerRunThunderbird"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 前置条件
Ubuntu18.04操作系统，已安装x11docker, 安装方法详见x11docker github仓库。    

### 2. 邮箱容器制作
撰写如下的Dockerfile:    

```
FROM x11docker/xfce
RUN sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y firefox-esr thunderbird libreoffice fonts-wqy-microhei fonts-wqy-zenhei xfonts-wqy thunderbird-l10n-zh-cn firefox-esr-l10n-zh-cn libreoffice-help-zh-cn manpages-zh
RUN apt-get install -y fontconfig
RUN apt-get install -y evince
RUN apt-get install -y fcitx-pinyin fonts-arphic-uming
RUN fc-cache -fv
COPY locale.gen /etc/locale.gen
RUN apt-get install -y fcitx-pinyin fonts-arphic-uming
RUN apt-get install -y tzdata
RUN  apt-get install -y locales tzdata xfonts-wqy && \
    locale-gen zh_CN.UTF-8 && \
    locale-gen  && \
    update-locale LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8 && \
    ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ENV LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8
```
其中`locale.gen`文件如下:      

```
zh_CN.UTF-8 UTF-8
```
运行以下命令编译一个名为`x11docker/securebrowser`的容器:     

```
$ docker build -t x11docker/securebrowser .
```
### 3. 容器启动及包装
撰写一个启动x11容器的命令文件:    

```
$ cat /home/xxx/start_en.sh
read id < <(x11docker --showid --network=host --home x11docker/securebrowser thunderbird)
docker exec -e XMODIFIERS="@im=fcitx" -e QT_IM_MODULE="fcitx" -e GTK_IM_MODULE="fcitx" $id fcitx&
```
撰写一个桌面快速启动文件以快速调用:     

```
$ cat /home/xxx/Secure.desktop
[Desktop Entry]
Version=1.0
Exec=xterm -e '/home/xxx/start_en.sh;sleep 10;bash'
Name=SecureApp
GenericName=SecureAPP
Comment=SecureApp
Encoding=UTF-8
Terminal=false
Type=Application
Categories=Application;Network;
```
注意需要安装`xterm`包以便可以方便的使用xterm调用启动文件。    
