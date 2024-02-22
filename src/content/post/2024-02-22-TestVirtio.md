+++
title= "TestVirtio"
date = "2024-02-22T19:46:22+08:00"
description = "TestVirtio"
keywords = ["Technology"]
categories = ["Technology"]
+++
创建:   

```
qemu-img create -f qcow2 testvirtio.qcow2 80G
```
挂载:    

![/images/2024_02_22_19_46_58_351x205.jpg](/images/2024_02_22_19_46_58_351x205.jpg)

进入SATA启动的win10系统后， 磁盘管理中可看(磁盘1, 对应F盘):   


![/images/2024_02_22_19_47_59_732x561.jpg](/images/2024_02_22_19_47_59_732x561.jpg)

关机，将SATA盘切换为virtio.    

切换前:    

![/images/2024_02_22_19_49_01_657x304.jpg](/images/2024_02_22_19_49_01_657x304.jpg)

添加:    

![/images/2024_02_22_19_49_27_440x207.jpg](/images/2024_02_22_19_49_27_440x207.jpg)

添加后:    

![/images/2024_02_22_19_49_40_522x411.jpg](/images/2024_02_22_19_49_40_522x411.jpg)

更换启动顺序:    

![/images/2024_02_22_19_50_26_493x439.jpg](/images/2024_02_22_19_50_26_493x439.jpg)

启动蓝屏:    

![/images/2024_02_22_19_50_39_897x661.jpg](/images/2024_02_22_19_50_39_897x661.jpg)

设备管理器中确实有virtio的驱动，但是启动时切换会蓝屏:    


![/images/2024_02_22_19_52_02_359x196.jpg](/images/2024_02_22_19_52_02_359x196.jpg)



