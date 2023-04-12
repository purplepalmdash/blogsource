+++
title= "WorkingTipsOnIGPUPassthroughOSX"
date = "2023-04-12T10:40:39+08:00"
description = "WorkingTipsOnIGPUPassthroughOSX"
keywords = ["Technology"]
categories = ["Technology"]
+++
Use diskutil for Change the EFI partition:    

![/images/2023_04_12_10_41_07_510x533.jpg](/images/2023_04_12_10_41_07_510x533.jpg)

Copy from iso's EFI to disk's EFI:     

```
% cp -r /Volumes/EFI/EFI /Volumes/EFI\ 1/
% sudo sync
% sudo shutdown -h now
```
Enable ssh:    

![/images/2023_04_12_10_45_08_525x420.jpg](/images/2023_04_12_10_45_08_525x420.jpg)

Then we could use ssh via:    

```
# ssh test@192.168.1.129
X11 forwarding request failed on channel 0
Last login: Wed Apr 12 10:45:16 2023 from 192.168.1.222
test@tests-iMac-Pro ~ % uname -a
Darwin tests-iMac-Pro.local 21.6.0 Darwin Kernel Version 21.6.0: Thu Mar  9 20:08:59 PST 2023; root:xnu-8020.240.18.700.8~1/RELEASE_X86_64 x86_64
```
Copy and replace EFI using igpu optimized version.   
 above 4G decoding forbid the hackintosh igpu passthrough.     
