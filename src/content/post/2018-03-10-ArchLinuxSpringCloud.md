+++
title = "WorkingTipsForArchLinuxSpring"
date = "2018-03-10T09:56:47+08:00"
description = "WorkingTipsForArchLinuxSpring"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Installation
Install the following packages:    

```
$ sudo pacman -S maven community/intellij-idea-community-edition
$ sudo pacman -S jdk8-openjdk jdk9-openjdk
$ sudo archlinux-java set java-9-openjdk
$ archlinux-java status
```
Since the intellij wants jdk8 or newer, you have to install newer jdk
implementation.   

Correct: the community edition didn't have the spring boot support, use the
ultimate edition:    

```
$ yaourt intellij-idea-ultimate-edition
```

### 扫盲
#### 什么是spring boot
spring boot 致力于简洁，让开发者写更少的配置，程序能够更快的运行和启动。它是下一代javaweb框架，并且它是spring cloud（微服务）的基础。


### spring boot
Create new project:    

![/images/2018_03_10_10_34_56_496x427.jpg](/images/2018_03_10_10_34_56_496x427.jpg)

Plugins:   

![/images/2018_03_10_10_38_10_469x284.jpg](/images/2018_03_10_10_38_10_469x284.jpg)


![/images/2018_03_10_13_32_55_802x411.jpg](/images/2018_03_10_13_32_55_802x411.jpg)

![/images/2018_03_10_13_33_53_774x425.jpg](/images/2018_03_10_13_33_53_774x425.jpg)


Import project: 

![/images/2018_03_10_15_36_18_639x257.jpg](/images/2018_03_10_15_36_18_639x257.jpg)

mvn aliyun configuration

![/images/2018_03_10_15_54_52_682x377.jpg](/images/2018_03_10_15_54_52_682x377.jpg)

In `/opt/maven/conf`.   

 
