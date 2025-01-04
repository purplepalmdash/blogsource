+++
title= "vfiogpuissue"
date = "2025-01-03T22:41:47+08:00"
description = "vfiogpuissue"
keywords = ["Technology"]
categories = ["Technology"]
+++
When `reboot` in vm, the host virsh command show this vm paused

```
root@idv:/var/lib/libvirt/images# virsh list
 Id   Name         State
---------------------------
 1    ubuntu2004   paused
```

logs:    

```
# tail -f /var/
char device redirected to /dev/pts/3 (label charserial0)
error: kvm run failed Bad address
RAX=ffff9f558281e020 RBX=ffff9f558281e000 RCX=0000000000000007 RDX=0000000000010101
RSI=ffff9f558281e004 RDI=0000000000000000 RBP=ffff9f558001b990 RSP=ffff9f558001b8e8
R8 =0000000000000001 R9 =ffffffffafaa6ee0 R10=ffff8f418fd18700 R11=0000000000000001
R12=0000000000010101 R13=0000000000cdcdcd R14=ffff9f558281e000 R15=ffff8f418fd18700
RIP=ffffffffaef257ae RFL=00000002 [-------] CPL=0 II=0 A20=1 SMM=0 HLT=0
ES =0000 0000000000000000 ffffffff 00800000
CS =0010 0000000000000000 ffffffff 00a09b00 DPL=0 CS64 [-RA]
SS =0018 0000000000000000 ffffffff 00c09300 DPL=0 DS   [-WA]
DS =0000 0000000000000000 ffffffff 00800000
FS =0000 00007fcd97123980 ffffffff 00800000
GS =0000 ffff8f41fbc00000 ffffffff 00800000
LDT=0000 0000000000000000 0000ffff 00000000
TR =0040 fffffe1b5fbfb000 00004087 00008b00 DPL=0 TSS64-busy
GDT=     fffffe1b5fbf9000 0000007f
IDT=     fffffe0000000000 00000fff
CR0=80050033 CR2=00007fcd981225a0 CR3=000000010ed44000 CR4=003506f0
DR0=0000000000000000 DR1=0000000000000000 DR2=0000000000000000 DR3=0000000000000000 
DR6=00000000ffff0ff0 DR7=0000000000000400
EFER=0000000000000d01
Code=fa 49 8d 76 04 44 21 c2 41 8b 3c 91 44 21 ef 89 fa 44 31 e2 <41> 89 16 85 c9 75 09 49 83 c7 01 b9 08 00 00 00 48 39 c6 75 c3 48 8b 45 b8 83 6d d0 01 4c

```

仅当连接有Hdmi的时候会发生这个现象。

