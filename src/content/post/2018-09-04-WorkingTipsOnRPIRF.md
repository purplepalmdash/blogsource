+++
title = "WorkingTipsOnRPIRF"
date = "2018-09-04T16:20:00+08:00"
description = "WorkingTipsOnRPIRF"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 连线图:    
RFID-RC522 board - Raspberry PI 1 Generation.   

```
SDA connects to Pin 24.
SCK connects to Pin 23.
MOSI connects to Pin 19.
MISO connects to Pin 21.
GND connects to Pin 6.
RST connects to Pin 22.
3.3v connects to Pin 1.
```
OR:    

```
(RC522) --- (GPIO RaspPi)
3.3v --- 1 (3V3)
SCK --- 23 (GPIO11)
MOSI --- 19 (GPIO10)
MISO --- 21 (GPIO09)
GND --- 25 (Ground)
RST --- 22 (GPIO25)
```
### RPI 配置 SPI
打开配置窗口:    

```
# sudo raspi-config
```
![/images/2018_09_04_16_45_35_866x313.jpg](/images/2018_09_04_16_45_35_866x313.jpg)

鼠标点击`5 Interfacing Options`, 选择`P4 SPI`:      

![/images/2018_09_04_16_46_07_839x233.jpg](/images/2018_09_04_16_46_07_839x233.jpg)

选择`Yes`后确认，打开rpi的SPI。    

重启后检查spi是否被正确加载:    

```
# lsmod | grep spi
spidev                  7034  0
spi_bcm2835             7424  0
```
### RPI Configuration
crontab中修改了`/bin/pdnsd/sh`文件，
`/etc/rc.local`文件中去掉了有关redsocks和pdnsd的选项。之后重启。     
### Python示例
可以参考：    

[https://pimylifeup.com/raspberry-pi-rfid-rc522/](https://pimylifeup.com/raspberry-pi-rfid-rc522/)    

### NodeJS例子

```
# mkdir RFID
# cd RFID
# wget http://node-arm.herokuapp.com/node_latest_armhf.deb
# dpkg -i node_latest_armhf.deb
```

