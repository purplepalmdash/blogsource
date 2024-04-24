+++
title= "ChangeItemsOnVentoy"
date = "2024-04-24T17:41:11+08:00"
description = "ChangeItemsOnVentoy"
keywords = ["Technology"]
categories = ["Technology"]
+++
Add custom menu after the default menu:    

```
root@vhdboot:/boot/efi/grub# diff grub.cfg grub.cfg.backback 
2664,2666d2663
< if [ -e $vt_plugin_path/ventoy/ventoy_grub.cfg ]; then
<     source $vt_plugin_path/ventoy/ventoy_grub.cfg
< fi
2675,2679c2672,2675
<     source $vt_plugin_path/ventoy/ventoy_grub.cfg
<     #menuentry "$NO_ISO_MENU (Press enter to reboot ...)" {
<     #    echo -e "\n    Rebooting ... "
<     #    reboot
<     #}
---
>     menuentry "$NO_ISO_MENU (Press enter to reboot ...)" {
>         echo -e "\n    Rebooting ... "
>         reboot
>     }
```
Ventoy configuration files:     

```
root@vhdboot:/boot/efi/grub# cat /mnt8/ventoy/ventoy.json 
{
    "control": [
	            { "VTOY_MENU_LANGUAGE": "zh_CN" },
		            { "VTOY_MENU_TIMEOUT": "3" },
        { "VTOY_DEFAULT_SEARCH_ROOT": "/HHHISO1" }
    ]
}
root@vhdboot:/boot/efi/grub# cat /mnt8/ventoy/ventoy_grub.cfg 
menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-d68c23a7-3d0b-4113-9127-8dac01ec1b29' {
	insmod gzio
	insmod part_gpt
	insmod ext2
	set root='hd0,gpt3'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt3 --hint-efi=hd0,gpt3 --hint-baremetal=ahci0,gpt3  d68c23a7-3d0b-4113-9127-8dac01ec1b29
	else
	  search --no-floppy --fs-uuid --set=root d68c23a7-3d0b-4113-9127-8dac01ec1b29
	fi
	linux	/boot/vmlinuz-6.5.0-28-generic root=UUID=d68c23a7-3d0b-4113-9127-8dac01ec1b29 ro  quiet splash $vt_handoff
	initrd	/boot/initrd.img-6.5.0-28-generic
}

menuentry "Boot Windows10" {    
    set my_vhd_path="/HHHISO/win10.vhdx"
    
    if search -n -s vdiskhd -f "$my_vhd_path"; then
        vhdboot_common_func "($vdiskhd)$my_vhd_path"
    else
        echo "$my_vhd_path not found"
    fi
}

menuentry "Boot Windows11" {    
    set my_vhd_path="/HHHISO/win11.vhdx"
    
    if search -n -s vdiskhd -f "$my_vhd_path"; then
        vhdboot_common_func "($vdiskhd)$my_vhd_path"
    else
        echo "$my_vhd_path not found"
    fi
}


menuentry 'Arch(linuxloop)' --class 'arch' {
	rmmod tpm
	img_path="/home/test/arch.img"
	img_uuid="6ab60fa1-d874-4b84-99d9-8ac0230f0303"
	search --no-floppy --set=root --file "${img_path}"
	loopback loop "${img_path}"
	linuxloops_args="rdinit=/linuxloops img_path=${img_path} img_uuid=${img_uuid}"
	export linuxloops_args
	if [ -f (loop,2)/grub2/grub.cfg ]; then
		configfile (loop,2)/grub2/grub.cfg
	else
		configfile (loop,2)/grub/grub.cfg
	fi
}

```
