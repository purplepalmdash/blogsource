+++
title= "USBDisplayLinkWorkingHowto"
date = "2020-09-15T14:07:38+08:00"
description = "USBDisplayLinkWorkingHowto"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Find USB Equipment
Via lsusb we could find which usb equipment available:     

```
$ sudo lsusb -t 
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/8p, 10000M
    |__ Port 5: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/16p, 480M
    |__ Port 2: Dev 2, If 0, Class=Vendor Specific Class, Driver=rtsx_usb, 480M
    |__ Port 3: Dev 3, If 0, Class=Vendor Specific Class, Driver=asix, 480M
    |__ Port 6: Dev 4, If 0, Class=Human Interface Device, Driver=, 12M
    |__ Port 6: Dev 4, If 1, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 9: Dev 5, If 0, Class=Hub, Driver=hub/4p, 480M
        |__ Port 1: Dev 7, If 1, Class=Human Interface Device, Driver=usbhid, 12M
        |__ Port 1: Dev 7, If 0, Class=Human Interface Device, Driver=usbhid, 12M
        |__ Port 2: Dev 9, If 0, Class=Human Interface Device, Driver=usbhid, 12M
        |__ Port 3: Dev 10, If 0, Class=Wireless, Driver=rndis_host, 480M
        |__ Port 3: Dev 10, If 1, Class=CDC Data, Driver=rndis_host, 480M
        |__ Port 4: Dev 12, If 0, Class=Vendor Specific Class, Driver=udl, 480M
    |__ Port 12: Dev 13, If 0, Class=Mass Storage, Driver=usb-storage, 480M
    |__ Port 13: Dev 6, If 1, Class=Video, Driver=uvcvideo, 480M
    |__ Port 13: Dev 6, If 0, Class=Video, Driver=uvcvideo, 480M
    |__ Port 14: Dev 8, If 0, Class=Wireless, Driver=btusb, 12M
    |__ Port 14: Dev 8, If 1, Class=Wireless, Driver=btusb, 12M
```
And we could found corresponding USB equipments:   

```
$ ls /sys/bus/usb/drivers/usb
1-12  1-13  1-14  1-2  1-3  1-6  1-9  1-9.1  1-9.2  1-9.3  1-9.4  2-5  bind  uevent  unbind  usb1  usb2
```
Via following command we could unbind/bind DisplayLink equipments:    

```
$ echo '1-9.4' | sudo tee /sys/bus/usb/drivers/usb/unbind
$ echo '1-9.4' | sudo tee /sys/bus/usb/drivers/usb/bind
```
`unbind` is equal to unplugin the usb port, `bind` is equal to insert the usb equipment.    

### xrandr tips
Via following commands:    

```
$ xrandr --output DVI-I-2-2  --mode 1920x1080 --right-of HDMI-0'
```
