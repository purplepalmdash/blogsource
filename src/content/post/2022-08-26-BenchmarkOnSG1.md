+++
title= "BenchmarkOnSG1"
date = "2022-08-26T11:09:33+08:00"
description = "BenchmarkOnSG1"
keywords = ["Technology"]
categories = ["Technology"]
+++
Start the redroid via:    

```
# docker run -itd --name redroid4cores --memory-swappiness=0 --privileged   -p 5556:5555 --cpuset-cpus 0-3 -v /root/cpus/present:/sys/devices/system/cpu/present -v /root/cpus/online:/sys/devices/system/cpu/online -v /root/cpus/possible:/sys/devices/system/cpu/possible redroid12:latest redroid.fps=120 ro.sf.lcd_density=240 redroid.width=1080 redroid.height=1920 redroid.gpu.mode=host redroid.gpu.node=/dev/dri/renderD128 androidboot.use_memfd=1
fe847de1c10fd9f28a358257b1c90c22884535a6d14c8867136bceea77cf140c
root@ctctest-UniServer-R4900-G3:~# docker exec -it redroid4cores sh
fe847de1c10f:/ # setprop sys.use_memfd 1 
```

![/images/2022_08_26_11_10_15_585x711.jpg](/images/2022_08_26_11_10_15_585x711.jpg)

Remove the other 3 socs:    

```
# rm -rf /dev/dri/renderD129 /dev/dri/renderD130 /dev/dri/renderD131 
# rm -rf /dev/dri/card1 /dev/dri/card2 /dev/dri/card3
```
Rerun the benchmark, still the same:    

![/images/2022_08_26_11_17_06_590x521.jpg](/images/2022_08_26_11_17_06_590x521.jpg)

Remove the device:    

```
# echo 1 > /sys/bus/pci/devices/0000:b8:00.0/remove
# echo 1 > /sys/bus/pci/devices/0000:c2:00.0/remove
# echo 1 > /sys/bus/pci/devices/0000:bd:00.0/remove
```
System hang.  

Using udev for removing the socs:    

```
# cat /etc/udev/rules.d/removesg1.rules 
ACTION=="add", KERNEL=="0000:b8:00.0", SUBSYSTEM=="pci", RUN+="/bin/sh -c 'echo 1 > /sys/bus/pci/devices/0000:b8:00.0/remove'"
ACTION=="add", KERNEL=="0000:bd:00.0", SUBSYSTEM=="pci", RUN+="/bin/sh -c 'echo 1 > /sys/bus/pci/devices/0000:bd:00.0/remove'"
ACTION=="add", KERNEL=="0000:c2:00.0", SUBSYSTEM=="pci", RUN+="/bin/sh -c 'echo 1 > /sys/bus/pci/devices/0000:c2:00.0/remove'"
```
Examine the graphical cards:    

```
# lspci | grep -i vga
0000:05:00.0 VGA compatible controller: ASPEED Technology, Inc. ASPEED Graphics Family (rev 41)
0000:b3:00.0 VGA compatible controller: Intel Corporation Device 4907 (rev 01)
```
render nodes:    

```
# ls /dev/dri/
by-path  card0  renderD128
# ls /dev/dri/by-path/ -l
total 0
lrwxrwxrwx 1 root root  8 8月  26 11:49 pci-0000:b3:00.0-card -> ../card0
lrwxrwxrwx 1 root root 13 8月  26 11:49 pci-0000:b3:00.0-render -> ../renderD128
```

Rerun the testing:     

![/images/2022_08_26_11_57_38_588x761.jpg](/images/2022_08_26_11_57_38_588x761.jpg)

### Another machine

![/images/2022_08_26_12_21_46_584x203.jpg](/images/2022_08_26_12_21_46_584x203.jpg)



Turbo boost mode(avd) comparision:    

non-turbo boost    

![/images/2022_08_26_12_35_07_372x255.jpg](/images/2022_08_26_12_35_07_372x255.jpg)


turbo boost(sg1):    

![/images/2022_08_26_12_41_30_373x417.jpg](/images/2022_08_26_12_41_30_373x417.jpg)

Turbo boot (t4):    

![/images/2022_08_26_12_48_07_525x494.jpg](/images/2022_08_26_12_48_07_525x494.jpg)

 vnc(best perf mode):     

![/images/2022_08_26_15_00_35_589x281.jpg](/images/2022_08_26_15_00_35_589x281.jpg)

no vnc(best perf mode), no scrcpy:     

![/images/2022_08_26_15_06_44_595x985.jpg](/images/2022_08_26_15_06_44_595x985.jpg)

scrcpy with 15fps:   

![/images/2022_08_26_15_22_06_528x261.jpg](/images/2022_08_26_15_22_06_528x261.jpg)

scrcpy with 5fps:   

![/images/2022_08_26_15_27_03_528x285.jpg](/images/2022_08_26_15_27_03_528x285.jpg)

no scrcpy again:   

![/images/2022_08_26_15_31_56_534x261.jpg](/images/2022_08_26_15_31_56_534x261.jpg)


t4 with scrcpy:    

![/images/2022_08_26_15_39_49_1029x473.jpg](/images/2022_08_26_15_39_49_1029x473.jpg)


ampere t4 with scrcpy:    

![/images/2022_08_26_15_49_37_480x918.jpg](/images/2022_08_26_15_49_37_480x918.jpg)

Performance:   

```
cpupower frequency-set -g performance

```
ampere t4 without scrcpy:   

![/images/2022_08_26_16_58_06_469x380.jpg](/images/2022_08_26_16_58_06_469x380.jpg)

should upgrade to oibaf?    upgrading to stable.   

```
sudo add-apt-repository ppa:kisak/kisak-mesa
sudo apt update
sudo apt upgrade
``` 
Next week, upgrading to 5.19.3

Even on 5.19.3, benchmark is very bad:    

![/images/2022_08_29_11_37_21_502x253.jpg](/images/2022_08_29_11_37_21_502x253.jpg)

