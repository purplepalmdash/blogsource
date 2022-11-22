+++
title= "WindowsVirtualBoxUCC"
date = "2022-10-26T09:19:03+08:00"
description = "WindowsVirtualBoxUCC"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Env Preparation
Install VirtualBox  7.0.2:    

![/images/2022_10_26_09_19_41_455x301.jpg](/images/2022_10_26_09_19_41_455x301.jpg)

View version:   

![/images/2022_10_26_09_20_32_843x589.jpg](/images/2022_10_26_09_20_32_843x589.jpg)

Disable the dhcp server for default `Host-only Networks`:    

![/images/2022_10_26_09_27_37_627x467.jpg](/images/2022_10_26_09_27_37_627x467.jpg)

### VM Creation
Create a new vm(type `Linux->Other Linux`):    

![/images/2022_10_26_09_29_24_778x373.jpg](/images/2022_10_26_09_29_24_778x373.jpg)

2 Cores, 2048MB Memory:    

![/images/2022_10_26_09_29_56_643x203.jpg](/images/2022_10_26_09_29_56_643x203.jpg)

Specify `centos85.vdi` file for hard disk:    

![/images/2022_10_26_09_30_38_648x273.jpg](/images/2022_10_26_09_30_38_648x273.jpg)

Configure the vm to specify its networking:    

![/images/2022_10_26_09_31_24_804x391.jpg](/images/2022_10_26_09_31_24_804x391.jpg)

Visit `https://192.168.56.199` should reach the ucc server web admin page:     

![/images/2022_10_26_09_34_44_771x540.jpg](/images/2022_10_26_09_34_44_771x540.jpg)

### UCC Client Testing
Create a new machine(type `Windows 10`):   

![/images/2022_10_26_09_36_04_747x326.jpg](/images/2022_10_26_09_36_04_747x326.jpg)

2 Cores, 3072 MB Memory, Enable EFI:    

![/images/2022_10_26_09_36_49_648x238.jpg](/images/2022_10_26_09_36_49_648x238.jpg)

Create 50 GB disk:    

![/images/2022_10_26_09_37_09_646x267.jpg](/images/2022_10_26_09_37_09_646x267.jpg)

Configure this VM, select `ICH9` as its chipset:   

![/images/2022_10_26_09_38_41_620x481.jpg](/images/2022_10_26_09_38_41_620x481.jpg)

Enable Network as its boot sequence:    

![/images/2022_10_26_09_40_04_594x370.jpg](/images/2022_10_26_09_40_04_594x370.jpg)

Change Network to `host-only` type:    

![/images/2022_10_26_09_42_16_713x344.jpg](/images/2022_10_26_09_42_16_713x344.jpg)

System will boot into EFI shell, input `exit` to bios:    

![/images/2022_10_26_09_40_52_603x279.jpg](/images/2022_10_26_09_40_52_603x279.jpg)

Select `Boot Manage`, press enter:    

![/images/2022_10_26_09_41_13_582x236.jpg](/images/2022_10_26_09_41_13_582x236.jpg)

Select `UEFI PXEv4` for booting:    

![/images/2022_10_26_09_41_34_638x346.jpg](/images/2022_10_26_09_41_34_638x346.jpg)

pxe will boot into register name, input name `ucctestwin10` for continue:    

![/images/2022_10_26_09_44_19_1152x854.jpg](/images/2022_10_26_09_44_19_1152x854.jpg)

Click setup button for initialization this node:    

![/images/2022_10_26_09_45_11_958x620.jpg](/images/2022_10_26_09_45_11_958x620.jpg)

Initialize the disk using the default parameters:    

![/images/2022_10_26_09_46_25_1706x560.jpg](/images/2022_10_26_09_46_25_1706x560.jpg)

Now return to login page, the `disk` and `network` indication should be green, enter username/password for continue:    

![/images/2022_10_26_09_49_10_470x605.jpg](/images/2022_10_26_09_49_10_470x605.jpg)

Click `TCI` for downloading the package and boot into windows 10:    

![/images/2022_10_26_09_49_59_1529x871.jpg](/images/2022_10_26_09_49_59_1529x871.jpg)

windows 10 in virtualbox:    

![/images/2022_10_26_09_59_21_909x816.jpg](/images/2022_10_26_09_59_21_909x816.jpg)

Don't try `idv`, cause in virtualbox it will run into error:    

![/images/2022_10_26_10_00_56_1024x531.jpg](/images/2022_10_26_10_00_56_1024x531.jpg)

### Ubuntu Verification
Upload the image and copy it into corresponding directory:    

![/images/2022_10_26_10_30_02_651x127.jpg](/images/2022_10_26_10_30_02_651x127.jpg)

Register image in UCC Server admin pages:    

![/images/2022_10_26_10_31_04_714x559.jpg](/images/2022_10_26_10_31_04_714x559.jpg)

Create a new machine for testing Ubuntu:    

![/images/2022_10_26_10_31_44_633x307.jpg](/images/2022_10_26_10_31_44_633x307.jpg)

2Core, 2048 MB , enable uefi:    

![/images/2022_10_26_10_32_08_663x260.jpg](/images/2022_10_26_10_32_08_663x260.jpg)

Select `ICH9` for chipset, and enable the network boot:    

![/images/2022_10_26_10_32_57_681x426.jpg](/images/2022_10_26_10_32_57_681x426.jpg)

Change the network:     

![/images/2022_10_26_10_33_13_568x366.jpg](/images/2022_10_26_10_33_13_568x366.jpg)

`exit` and into the bios:   

![/images/2022_10_26_10_34_06_616x290.jpg](/images/2022_10_26_10_34_06_616x290.jpg)

Boot Manager and select `UEFI PXEv4`:    

![/images/2022_10_26_10_34_33_600x419.jpg](/images/2022_10_26_10_34_33_600x419.jpg)

Register client:    

![/images/2022_10_26_10_35_33_658x218.jpg](/images/2022_10_26_10_35_33_658x218.jpg)

Initialize the disk:    

![/images/2022_10_26_10_36_03_1413x407.jpg](/images/2022_10_26_10_36_03_1413x407.jpg)

Select Ubuntu for downloading and boot into system:    

![/images/2022_10_26_10_37_38_1001x301.jpg](/images/2022_10_26_10_37_38_1001x301.jpg)

Ubuntu desktop:    

![/images/2022_10_26_10_41_47_809x720.jpg](/images/2022_10_26_10_41_47_809x720.jpg)


