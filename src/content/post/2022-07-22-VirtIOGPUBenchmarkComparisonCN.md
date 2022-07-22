+++
title= "VirtIOGPUBenchmarkComparisonCN"
date = "2022-07-22T15:42:09+08:00"
description = "VirtIOGPUBenchmarkComparisonCN"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 硬件信息
内存速度通过`sudo dmidecode --type 17`来检测    
平台 1(i7 9代机器, 4核8线程，32G内存):    

```
CPU: Intel(R) Core(TM) i7-9700K CPU @ 3.60GHz
Memory: MemTotal: 32709264 kB, 2666 MT/s
```

平台 2(i5 4代机器，4核4线程，32G内存):    

```
CPU: Intel(R) Core(TM) i5-4460  CPU @ 3.20GHz
Memory: MemTotal: 32808212 kB, 1600 MT/s 
```
测试中使用同一块sata ssd:   

```
Disk 1: Crucial 500GB sata ssd. 
```
测试显卡信息, 一块为WX5100 8G, 一块为5700 xt 8G(检测命令为 `lspci | grep -i vga`):    

```
Card1: VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 [Radeon RX 5600 OEM/5600 XT / 5700/5700 XT] (rev c4)
Card2: VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon Pro WX 5100]
```
### 2. 系统/软件信息
列举如下:   

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
软件信息:    

```
Unigine_Heaven-4.0
Unigine_Valley-1.0
glmark2
Redroid(Android 12)
scrcpy
```
### 3. 桌面场景测试
| 测试样例 | 目的描述 | 简要结论 |
| ----------- | -------------- | ----------- |
| 3.1 裸机对比(WX5100)  |裸机opengl及游戏场景对比-WX5100 | |
| 3.2 裸机对比(5700xt)  |裸机opengl及游戏场景对比-5700xt | |
| 3.3 Virgl On i5-4460(WX5100) | Virgl 性能对比 - i5-4460+WX5100 | |
| 3.4 Virgl On i7-9700K(WX5100) | Virgl 性能对比 - i7-9700K+WX5100 | |
| 3.5 Virgl On i5-4460(5700xt) | Virgl 性能对比 - i5-4460+5700xt | |
| 3.6 Virgl On i7-9700K(5700xt) | Virgl 性能对比 - i7-9700K+5700xt | |

#### 3.1 裸机对比(WX5100)
|  | i5-4460 | i7-9700K |
| ----------- | -------------- | ----------- |
| glmark2  | 9140, 9117, 9117 | 9730, 9651, 9660 |
| Unigine_Valley FPS  | 43, 42.4, 42.3 | 45.6, 45.0, 45.0 |
| Unigine_Valley Score  | 1800, 1773, 1769 | 1906, 1884, 1884 |
| Unigine_Heaven FPS  | 44.8 | 46.9 |
| Unigine_Heaven Score  | 1130 | 1182 |


结论:    
1. i5/i7平台上，WX5100表现基本一致。
2. 因WX5100算力比较一案，i5-4代没有成为测试中的瓶颈位。

#### 3.2 裸机对比(5700xt)
|  | i5-4460 | i7-9700K |
| ----------- | -------------- | ----------- |
| glmark2  | 13950, 14165, 14315  | 22134, 22256, 22150 |
| Unigine_Valley FPS  | 105, 103.1, 102.9 | 168.7, 169.1, 168.5 |
| Unigine_Valley Score  | 4391, 4315, 4304 | 7057, 7068, 7043 |
| Unigine_Heaven FPS  | 167.0 | 171.1 |
| Unigine_Heaven Score  | 4206 | 4311 |
| Unigine_Heaven FPS(4k)  | 43 | 42.8 |
| Unigine_Heaven Score(4k)  | 1083 | 1077 |


结论:    
1. 5700xt在i7平台上运行远比i5平台上流畅，4代i5平台无法发挥5700xt的算力特性，具体表现为：glmark2跑分差距较大，Unigine valley帧率相差较大(1080场景下掉了约70帧)，4K下表现一致。
2. 显卡选型时需充分考虑CPU/内存等平台算力特性。  

 
#### 3.3 Virgl On i5-4460(WX5100)

|  | i5-4460 | Virgl on i5-4460 |
| ----------- | -------------- | ----------- |
| glmark2  | 9140, 9117, 9117 | 2892, 2859, 2827 |
| Unigine_Valley FPS  | 43, 42.4, 42.3 | 40.8, 40.5, 40.6 |
| Unigine_Valley Score  | 1800, 1773, 1769 | 1707, 1694, 1699 |
| Unigine_Heaven FPS  | 44.8 | 42.2 |
| Unigine_Heaven Score  | 1130 | 1062 |


结论:    
1. glmark2: Virgl跑分约为裸机的31.3%。   
2. Unigine场景跑分: Virgl性能并未出现明显折损，因为WX5100本身算力一般, CPU足够支撑virgl转码开销。    

#### 3.4 Virgl On i7-9700K(WX5100)

|  | i7-9700K | Virgl on i7-9700K |
| ----------- | -------------- | ----------- |
| glmark2  | 9730, 9651, 9660 | 4274, 4262, 4320 |
| Unigine_Valley FPS  | 45.6, 45.0, 45.0 | 42.3, 41.6, 41.5 |
| Unigine_Valley Score  | 1906, 1884, 1884 | 1770, 1740, 1737 |
| Unigine_Heaven FPS  | 46.9 | 41.9 | 
| Unigine_Heaven Score  | 1182 | 1055 |

结论:   
1. glmark2: Virgl 约为裸机跑分的44.2%。    
2. Unigine场景跑分: Virgl性能并未出现明显折损，因为WX5100本身算力一般, CPU足够支撑virgl转码开销。    
3. 提升平台算力(CPU,内存等)有助于提升opengl转码效率(对比3.3场景下glmark2跑分).    

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


结论:   
1. glmark2: Virgl约为裸机的21%。   
2. Unigine Valley/Unigine Heaven: Virgl约为裸机跑分的55%~60%, 但在4K场景下，Virgl为裸机的87%。 

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


结论:   
1. glmark2: Virgl约为裸机的19%。   
2. Unigine Valley/Unigine Heaven: Virgl约为裸机跑分的55%~60%, 但在4K场景下，Virgl为裸机的83%。 

### 4. Android 场景
| 硬件平台 | 跑分(开启scrcpy) | 跑分(不开启scrcpy) |
| ----------- | -------------- | ----------- |
| i7-9700K+5700xt  | 616521(剑侠山谷: 309760 , 禅院悟道: 306761) | 617133(剑侠山谷: 310050, 禅院悟道: 307083) |
| Virgl on (i7-9700K+5700xt)  | 442043(剑侠山谷: 223850 , 禅院悟道: 218193) | 564343(剑侠山谷: 289202, 禅院悟道:275141) |
| i5-4460+5700xt  | 384564(剑侠山谷: 112763 , 禅院悟道: 221801) | 592030(剑侠山谷: 283705, 禅院悟道: 308255) |
| Virgl on (i5-4460+5700xt)  | 227747(剑侠山谷: 108206 , 禅院悟道: 119541) | 445234(剑侠山谷: 214831, 禅院悟道:230403) |
| i7-9700K+WX5100  | 567503(剑侠山谷: 265995 , 禅院悟道: 301508) | 585720(剑侠山谷: 277133, 禅院悟道: 308587) |
| Virgl on (i7-9700K+WX5100)  | 412764(剑侠山谷: 210150 , 禅院悟道: 202614) | 499942(剑侠山谷: 257037, 禅院悟道: 242905) |
| i5-4460+WX5100  | 401626(剑侠山谷: 166695 , 禅院悟道: 234931) | 572952(剑侠山谷: 267744, 禅院悟道: 305208) |
| Virgl on (i5-4460+WX5100)  | 225837(剑侠山谷: 106071 , 禅院悟道: 119766) | 445202(剑侠山谷: 211450, 禅院悟道:233752) |


结论:   
1. 鲁大师跑分极限: 约为60+万分，跑分时支持的最大分辨率为1080p(待确认)?   
2. scrcpy开启与否会影响跑分结果: 尤其在低端CPU上跑分下降尤为明显。   
3. 当GPU算力不至于成为瓶颈的场景下，virgl在安卓下渲染的跑分可以达到裸机极限跑分的: 90% (i7-9700K + 5700xt)   
4. WX5100可能成为跑分的性能瓶颈的场景下， virgl安卓渲染跑分在i7下可达到裸机极限的: 85% (i7-9700K + WX5100), i5下可达到裸机极限的: 77% (i5-4460 + WX5100)
