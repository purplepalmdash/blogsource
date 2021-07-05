+++
title= "WorkingTipsOnMultiSeat"
date = "2021-07-06T06:07:44+08:00"
description = "WorkingTipsOnMultiSeat"
keywords = ["Technology"]
categories = ["Technology"]
+++
### steps
default seat:    

```
# loginctl seat-status seat0>seat0.txt
# cat seat0.txt
seat0
	Sessions: *1
	 Devices:
		  ├─/sys/devices/LNXSYSTM:00/LNXPWRBN:00/input/input1
		  │ input:input1 "Power Button"
		  ├─/sys/devices/LNXSYSTM:00/LNXSYBUS:00/PNP0A08:00/LNXVIDEO:00/input/input14
		  │ input:input14 "Video Bus"
		  ├─/sys/devices/LNXSYSTM:00/LNXSYBUS:00/PNP0C0C:00/input/input0
		  │ input:input0 "Power Button"
		  ├─/sys/devices/pci0000:00/0000:00:02.0/drm/card0
		  │ [MASTER] drm:card0
		  │ ├─/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-DP-1
		  │ │ [MASTER] drm:card0-DP-1
		  │ ├─/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-HDMI-A-1
		  │ │ [MASTER] drm:card0-HDMI-A-1
		  │ ├─/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-HDMI-A-2
		  │ │ [MASTER] drm:card0-HDMI-A-2
		  │ └─/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-eDP-1
		  │   [MASTER] drm:card0-eDP-1
		  │   └─/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-eDP-1/intel_backlight
		  │     backlight:intel_backlight
		  ├─/sys/devices/pci0000:00/0000:00:02.0/graphics/fb0
		  │ graphics:fb0 "i915drmfb"
		  ├─/sys/devices/pci0000:00/0000:00:03.0/sound/card1
		  │ sound:card1 "HDMI"
		  │ ├─/sys/devices/pci0000:00/0000:00:03.0/sound/card1/input16
		  │ │ input:input16 "HDA Intel HDMI HDMI/DP,pcm=3"
		  │ ├─/sys/devices/pci0000:00/0000:00:03.0/sound/card1/input17
		  │ │ input:input17 "HDA Intel HDMI HDMI/DP,pcm=7"
		  │ ├─/sys/devices/pci0000:00/0000:00:03.0/sound/card1/input18
		  │ │ input:input18 "HDA Intel HDMI HDMI/DP,pcm=8"
		  │ ├─/sys/devices/pci0000:00/0000:00:03.0/sound/card1/input19
		  │ │ input:input19 "HDA Intel HDMI HDMI/DP,pcm=9"
		  │ └─/sys/devices/pci0000:00/0000:00:03.0/sound/card1/input20
		  │   input:input20 "HDA Intel HDMI HDMI/DP,pcm=10"
		  ├─/sys/devices/pci0000:00/0000:00:14.0/usb2
		  │ usb:usb2
		  │ └─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3
		  │   usb:2-3
		  │   └─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3/2-3:1.2/0003:046D:C52B.0003/0003:046D:404D.0004/input/input15
		  │     input:input15 "Logitech K400 Plus"
		  │     ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3/2-3:1.2/0003:046D:C52B.0003/0003:046D:404D.0004/input/input15/input15::capslock
		  │     │ leds:input15::capslock
		  │     ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3/2-3:1.2/0003:046D:C52B.0003/0003:046D:404D.0004/input/input15/input15::compose
		  │     │ leds:input15::compose
		  │     ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3/2-3:1.2/0003:046D:C52B.0003/0003:046D:404D.0004/input/input15/input15::kana
		  │     │ leds:input15::kana
		  │     ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3/2-3:1.2/0003:046D:C52B.0003/0003:046D:404D.0004/input/input15/input15::numlock
		  │     │ leds:input15::numlock
		  │     └─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3/2-3:1.2/0003:046D:C52B.0003/0003:046D:404D.0004/input/input15/input15::scrolllock
		  │       leds:input15::scrolllock
		  ├─/sys/devices/pci0000:00/0000:00:14.0/usb3
		  │ usb:usb3
		  ├─/sys/devices/pci0000:00/0000:00:1b.0/sound/card0
		  │ sound:card0 "PCH"
		  │ └─/sys/devices/pci0000:00/0000:00:1b.0/sound/card0/input8
		  │   input:input8 "HDA Intel PCH Headphone"
		  ├─/sys/devices/pci0000:00/0000:00:1d.0/usb1
		  │ usb:usb1
		  │ └─/sys/devices/pci0000:00/0000:00:1d.0/usb1/1-1
		  │   usb:1-1
		  └─/sys/devices/platform/pcspkr/input/input7
		    input:input7 "PC Speaker"
```
找寻对应的口：    

```
/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-eDP-1
/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3

/sys/devices/pci0000:00/0000:00:14.0/usb2/2-3

/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-HDMI-A-1

```

Attach:    

```
# loginctl attach seat1  /sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-eDP-1
# loginctl attach seat1  /sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3
➜  ~ loginctl seat-status seat1
seat1
	 Devices:
		  ├─/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-eDP-1
		  │ [MASTER] drm:card0-eDP-1
		  │ └─/sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-eDP-1/intel_backlight
		  │   backlight:intel_backlight
		  └─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3
		    usb:2-2.3
		    ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.0/0003:1A81:2019.0005/input/input10
		    │ input:input10 "G-Tech Fuhlen SM680 Mechanical Keyboard"
		    │ ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.0/0003:1A81:2019.0005/input/input10/input10::capslock
		    │ │ leds:input10::capslock
		    │ ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.0/0003:1A81:2019.0005/input/input10/input10::compose
		    │ │ leds:input10::compose
		    │ ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.0/0003:1A81:2019.0005/input/input10/input10::kana
		    │ │ leds:input10::kana
		    │ ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.0/0003:1A81:2019.0005/input/input10/input10::numlock
		    │ │ leds:input10::numlock
		    │ └─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.0/0003:1A81:2019.0005/input/input10/input10::scrolllock
		    │   leds:input10::scrolllock
		    ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.1/0003:1A81:2019.0006/input/input11
		    │ input:input11 "G-Tech Fuhlen SM680 Mechanical Keyboard Mouse"
		    ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.1/0003:1A81:2019.0006/input/input12
		    │ input:input12 "G-Tech Fuhlen SM680 Mechanical Keyboard"
		    ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.1/0003:1A81:2019.0006/input/input13
		    │ input:input13 "G-Tech Fuhlen SM680 Mechanical Keyboard Consumer Control"
		    ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.1/0003:1A81:2019.0006/input/input14
		    │ input:input14 "G-Tech Fuhlen SM680 Mechanical Keyboard System Control"
		    ├─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.2/0003:1A81:2019.0007/input/input16
		    │ input:input16 "G-Tech Fuhlen SM680 Mechanical Keyboard"
		    └─/sys/devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2.3/2-2.3:1.3/0003:1A81:2019.0008/input/input17
		      input:input17 "G-Tech Fuhlen SM680 Mechanical Keyboard"
```
Verify:    

```
➜  ~ ls -l  /etc/udev/rules.d/
total 12
-rw-r--r-- 1 root root  76 Jul  6 06:22 72-seat-drm-pci-0000_00_02_0.rules
-rw-r--r-- 1 root root  86 Jul  6 06:23 72-seat-usb-pci-0000_00_14_0-usb-0_2_3.rules
-rw-r--r-- 1 root root 432 Aug 10  2020 99-kvmd.rules.pacsave

```
Disable lxdm and testing:    

```
# systemctl disable lxdm
Removed /etc/systemd/system/display-manager.service.
# reboot
```
