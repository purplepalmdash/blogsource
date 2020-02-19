+++
title = "把SurfacePro作为外置显示器"
date = "2018-06-21T16:45:10+08:00"
description = "SurfacePro"
keywords = ["Linux"]
categories = ["Technology"]
+++
2014年我花了3688买的SurfacePro一代，一直以来都处于准吃灰状态。当你习惯了ArchLinux+Awesome的快捷以后，恐怕市面上所有主流操作系统都不会入你法眼。当某个桌面操作需要你用到鼠标的时候，事实上你已经输掉了时间----只有双手不离开键盘完成某个窗口的切换或是缩放，才是真正的高效，这也是Awesome之所以成为我首选桌面环境的一大原因。    

效率上去了，然而屏幕永远是不够的，笔记本本身的1600x900之外，我在DP口外挂了一块1080P的显示器，如果需要同时在几个开发环境下倒腾，就有点捉襟见肘了。所以后面我买了块DisplayLink的USB显卡，又找来另一个1080P的显示器，勉强实现了三个工作桌面。但是其缺点也是明显的：Display Link在Linux下并不是特别稳定，窗口拖影也比较严重。前段时间调整座位，显示器被主人拿回。瞬间又回到了两屏工作的悲惨境地。于是我动起了SurfacePro的脑筋。    

### 物料准备
我准备的物料如下:    

```
Surface Pro    
100M USB网卡x2
千兆交换机(其实百兆就够了，毕竟网卡也是100M的）
网线两根
```
### 系统配置
HP 8460P , ArchLinux.    
Surface Pro, Windows 10 x86_64.    

### Linux配置
ArchLinux下，安装x11vnc:   

```
# sudo pacman -S x11vnc
```
添加一个假的显示器.    

在命令行里运行:    

```
$ gtf 1920 1080 60

  # 1920x1080 @ 60.00 Hz (GTF) hsync: 67.08 kHz; pclk: 172.80 MHz
  Modeline "1920x1080_60.00"  172.80  1920 2040 2248 2576  1080 1081 1084 1118  -HSync +Vsync
```
这是为了获得显示屏的配置参数，拷贝`Modeline "1920x1080_60.00"
`以后的部分到剪切版。接下来我们将使用这一堆值来设置一个新的显示模式.    

```
$ xrandr --newmode "1920x1080_60.00" 172.80  1920 2040 2248 2576  1080 1081 1084 1118  -HSync +Vsync
$ xrandr --addmode VIRTUAL1 1920x1080_60.00
$ xrandr --output VIRTUAL1 --mode 1920x1080_60.00 --left-of LVDS1
```
以上三行配置了`VIRTUAL1`屏幕，位于笔记本屏幕的左边。事实上我的屏幕摆放如下:    

![/images/2018_06_21_17_00_36_706x428.jpg](/images/2018_06_21_17_00_36_706x428.jpg)

现在启动x11vnc, 如下:    

```
# x11vnc -noxdamage -clip  1920x1080+0+0
```
插入USB网卡，配置网络:    

```
# sudo ifconfig enp37s0u2 10.78.79.3
```
### SurfacePro配置
插入USB网卡，配置网络，    

```
IP： 10.78.79.2
子网掩码: 255.255.255.0
```
从`http://www.uvnc.com/docs/uvnc-viewer.html`下载UltraVNC Viewer,
运行，并访问`10.78.79.3:5900`:    

![/images/2018_06_21_17_04_54_389x380.jpg](/images/2018_06_21_17_04_54_389x380.jpg)

即进入到我们创建出来的VIRTUAL1屏幕。    


将UltraVNC全屏，并隐藏菜单栏，现在你的SurfacePro上应该显示出一个新的工作空间，第三块工作桌面挂载完毕。三屏工作情况下，效率又可以上到一个新的台阶。    

![/images/2018_06_21_17_08_34_701x521.jpg](/images/2018_06_21_17_08_34_701x521.jpg)

### 未来改进
1. 1920x1080的屏幕分辨率在100M网络上传输还是会有少许延迟。可以改为1600x900的分辨率。   
2. 可以考虑通过USB-USB的连接方式, USB Tethering,
   应该可以达到USB2.0甚至3.0的连接速度?         

来说说优点:   
8460P本身仅支持双屏显示，然而用这种x11vnc的方式，理论上可以支持无限多块外挂屏幕。

### 改进
2019.03.13    

用到了usb 2.0的网卡，实际上有100M带宽，用来跑x11vnc速度还是很慢。    

后面我找到了用tigervnc的x0vncserver方式。    


```
x0vncserver -Geometry 1080x1920+0+0 -SecurityTypes=none
```

