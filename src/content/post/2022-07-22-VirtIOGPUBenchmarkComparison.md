+++
title= "VirtIOGPUBenchmarkComparison"
date = "2022-07-22T11:27:32+08:00"
description = "VirtIOGPUBenchmarkComparison"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. HardWare Info
Detect the memory speed via `sudo dmidecode --type 17`.    
Hardware 1(i7 9th machine):    

```
CPU: Intel(R) Core(TM) i7-9700K CPU @ 3.60GHz
Memory: MemTotal: 32709264 kB, 2666 MT/s
```

Hardware 2(i5 machine):    

```
CPU: Intel(R) Core(TM) i5-4460  CPU @ 3.20GHz
Memory: MemTotal: 32808212 kB, 1600 MT/s 
```
Use the same ssd disk:   

```
Disk 1: Crucial 500GB sata ssd. 
Disk 2: nvme ssd 256GB. 
```
Graphical Card Info(Detected via `lspci | grep -i vga`):    

```
Card1: VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 [Radeon RX 5600 OEM/5600 XT / 5700/5700 XT] (rev c4)
Card2: VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon Pro WX 5100]
```
### 2. System/SoftWare Info
System and Software info listed as:    

500GB Crucial ssd:   

```
$ cat /etc/issue
Ubuntu 20.04.4 LTS \n \l
$ uname -a
Linux hope 5.4.0-120-generic #136-Ubuntu SMP Fri Jun 10 13:40:48 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
$ qemu-system-x86_64 --version
QEMU emulator version 6.0.0 (Debian 1:6.0+dfsg-2expubuntu1~focal1.0)
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
$ dpkg -l | grep -i virgl
ii  libvirglrenderer-dev:amd64                 0.8.2-1ubuntu1.1                      amd64        virtual GPU for KVM virtualization - headers
ii  libvirglrenderer1:amd64                    0.8.2-1ubuntu1.1                      amd64        virtual GPU for KVM virtualization
```
nvme ssd 256GB:    

```
$ cat /etc/issue
Pop!_OS 21.10 \n \l
$ uname -a
Linux pop-os 5.17.15-76051715-generic #202206141358~1655919116~21.10~1db9e34 SMP PREEMPT Wed Jun 22 19 x86_64 x86_64 x86_64 GNU/Linux
$ qemu-system-x86_64 --version
QEMU emulator version 6.0.0 (Debian 1:6.0+dfsg-2expubuntu1.3)
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
$ dpkg -l | grep -i virgl
ii  libvirglrenderer1:amd64                 0.8.2-5ubuntu0.21.10.1                                            amd64        virtual GPU for KVM virtualization
```
Software:    

```
Unigine_Heaven-4.0
Unigine_Valley-1.0
glmark2
Redroid(Android 12)
```
### 3. Desktop Scenario
| Case Lists | Description | Conclusion |
| ----------- | -------------- | ----------- |
| 3.1 BareMetal Comparison(WX5100)  |BareMetal Performance comparison using WX5100 | |
| 3.2 BareMetal Comparison(5700xt)  |BareMetal Performance comparison using 5700xt | |
| 3.3 Virgl On i5-4460(WX5100) | Virgl Performance on i5-4460 using WX5100 | |
| 3.4 Virgl On i7-9700K(WX5100) | Virgl Performance on i7-9700K using WX5100 | |
| 3.5 Virgl On i5-4460(5700xt) | Virgl Performance on i5-4460 using 5700xt | |
| 3.6 Virgl On i7-9700K(5700xt) | Virgl Performance on i7-9700K using 5700xt | |

#### 3.1 BareMetal Comparison(WX5100)
|  | i5-4460 | i7-9700K |
| ----------- | -------------- | ----------- |
| glmark2  | 9140, 9117, 9117 | 9730, 9651, 9660 |
| Unigine_Valley FPS  | 43, 42.4, 42.3 | 45.6, 45.0, 45.0 |
| Unigine_Valley Score  | 1800, 1773, 1769 | 1906, 1884, 1884 |
| Unigine_Heaven FPS  | 44.8 | 46.9 |
| Unigine_Heaven Score  | 1130 | 1182 |


Conclusion:    
1. WX5100 behaves almost the same on both i5/i7 platforms.   
2. i5's cpu power won't be a bottleneck in WX5100 scenario.  

#### 3.2 BareMetal Comparison(5700xt)
|  | i5-4460 | i7-9700K |
| ----------- | -------------- | ----------- |
| glmark2  | 13950, 14165, 14315  | 22134, 22256, 22150 |
| Unigine_Valley FPS  | 105, 103.1, 102.9 | 168.7, 169.1, 168.5 |
| Unigine_Valley Score  | 4391, 4315, 4304 | 7057, 7068, 7043 |
| Unigine_Heaven FPS  | 167.0 | 171.1 |
| Unigine_Heaven Score  | 4206 | 4311 |
| Unigine_Heaven FPS(4k)  | 43 | 42.8 |
| Unigine_Heaven Score(4k)  | 1083 | 1077 |


Conclusion:    
1. 5700xt runs smoother in i7 than on i5, perhaps i5 4th is too old for working together with 5700xt?(4 cores, bus width, etc).     
2. i5's CPU/Memory becomes a bottleneck in 5700xt testing.    

 
#### 3.3 Virgl On i5-4460(WX5100)

|  | i5-4460 | Virgl on i5-4460 |
| ----------- | -------------- | ----------- |
| glmark2  | 9140, 9117, 9117 | 2892, 2859, 2827 |
| Unigine_Valley FPS  | 43, 42.4, 42.3 | 40.8, 40.5, 40.6 |
| Unigine_Valley Score  | 1800, 1773, 1769 | 1707, 1694, 1699 |
| Unigine_Heaven FPS  | 44.8 | 42.2 |
| Unigine_Heaven Score  | 1130 | 1062 |


Conclusion:    
1. glmark2: Virgl is 31.3% of the baremetal performance.     
2. Unigine Valley/Unigine Heaven: Virgl performance almost the same as baremetal.   

#### 3.4 Virgl On i7-9700K(WX5100)

|  | i7-9700K | Virgl on i7-9700K |
| ----------- | -------------- | ----------- |
| glmark2  | 9730, 9651, 9660 | 4274, 4262, 4320 |
| Unigine_Valley FPS  | 45.6, 45.0, 45.0 | 42.3, 41.6, 41.5 |
| Unigine_Valley Score  | 1906, 1884, 1884 | 1770, 1740, 1737 |
| Unigine_Heaven FPS  | 46.9 | 41.9 | 
| Unigine_Heaven Score  | 1182 | 1055 |

Conclusion:    
1. glmark2: Virgl is 44.2% of the baremetal performance.     
2. Unigine Valley/Unigine Heaven: Virgl performance almost the same as baremetal.   
3. Comparing to 3.3: glmark2 will be improved using powerful CPU/Memory.   

#### 3.5 Virgl On i5-4460(5700xt)

|  | i5-4460 | Virgl on i5-4460 |
| ----------- | -------------- | ----------- |
| glmark2  | 13950, 14165, 14315  | 2916, 3174, 2832 |
| Unigine_Valley FPS  | 105, 103.1, 102.9 | 61, 61.9, 60.9 | 
| Unigine_Valley Score  | 4391, 4315, 4304 | 2551, 2588, 2548 |
| Unigine_Heaven FPS  | 167.0 | 89.7 |
| Unigine_Heaven Score  | 4206 | 2271 | 
| Unigine_Heaven FPS(4k)  | 43 | 37.8 |
| Unigine_Heaven Score(4k)  | 1083 | 951 |


Conclusion:    
1. glmark2: Virgl is 21% of the baremetal performance.     
2. Unigine Valley/Unigine Heaven: Virgl is around 55%~60% of the baremetal performance, while in 4k behavior, 87% of the baremetal performance. 

#### 3.6 Virgl On i7-9700K(5700xt)

|  | i7-9700K | Virgl on i7-9700K |
| ----------- | -------------- | ----------- |
| glmark2  | 22134, 22256, 22150 | 4152, 4256, 4238 |
| Unigine_Valley FPS  | 168.7, 169.1, 168.5 | 91.6, 91.5, 91.4 | 
| Unigine_Valley Score  | 7057, 7068, 7043 | 3831, 3829, 3825 |
| Unigine_Heaven FPS  | 171.1 | 85.3 | 
| Unigine_Heaven Score  | 4311 | 2149 |
| Unigine_Heaven FPS(4k)  | 42.8 |  35.6 | 
| Unigine_Heaven Score(4k)  | 1077 | 896 |


Conclusion:    
1. glmark2: Virgl is 19% of the baremetal performance.     
2. Unigine Valley/Unigine Heaven: Virgl is around 55%~60% of the baremetal performance, while in 4k behavior, 83% of the baremetal performance. 

### 4. Android Scenario
| Hardware/Platform | Score(scrcpy) | Score(no scrcpy) |
| ----------- | -------------- | ----------- |
| i7-9700K+5700xt  | 616521(剑侠山谷: 309760 , 禅院悟道: 306761) | 617133(剑侠山谷: 310050, 禅院悟道: 307083) |
| Virgl on (i7-9700K+5700xt)  | 442043(剑侠山谷: 223850 , 禅院悟道: 218193) | 564343(剑侠山谷: 289202, 禅院悟道:275141) |
| i5-4460+5700xt  | 384564(剑侠山谷: 112763 , 禅院悟道: 221801) | 592030(剑侠山谷: 283705, 禅院悟道: 308255) |
| Virgl on (i5-4460+5700xt)  | 227747(剑侠山谷: 108206 , 禅院悟道: 119541) | 445234(剑侠山谷: 214831, 禅院悟道:230403) |
| i7-9700K+WX5100  | 567503(剑侠山谷: 265995 , 禅院悟道: 301508) | 585720(剑侠山谷: 277133, 禅院悟道: 308587) |
| Virgl on (i7-9700K+WX5100)  | 412764(剑侠山谷: 210150 , 禅院悟道: 202614) | 499942(剑侠山谷: 257037, 禅院悟道: 242905) |
| i5-4460+WX5100  | 401626(剑侠山谷: 166695 , 禅院悟道: 234931) | 572952(剑侠山谷: 267744, 禅院悟道: 305208) |
| Virgl on (i5-4460+WX5100)  | 225837(剑侠山谷: 106071 , 禅院悟道: 119766) | 445202(剑侠山谷: 211450, 禅院悟道:233752) |


Conclusion:    
1. Ludashi's top score: around 61W, the maximum performance would be 1080p?   
2. scrcpy will greatly impact the performance when using Virgl  under low-end CPU.   
3. virgl will be 85% for the best of bare metal on WX5100 on i7, 91% of bare metal on 5700xt on i7
