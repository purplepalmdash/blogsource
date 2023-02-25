+++
title= "vtdlibvirtconfiguration"
date = "2023-02-24T10:36:11+08:00"
description = "vtdlibvirtconfiguration"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
Install kde desktop environment:    

```
$ sudo apt install -y kubuntu-desktop
```
Install libvirtd and remove default qemu, then install compiled qemu:    

```
$  sudo apt install -y libvirt-bin
$  sudo apt remove qemu-system-x86
// Go to   qemu source code: 
#  make install 
# qemu-system-x86_64 --version
QEMU emulator version 4.2.0 (v4.2.0-dirty)
Copyright (c) 2003-2019 Fabrice Bellard and the QEMU Project developers
```
Install virt-manager for managing the vms:     

```
$ sudo apt install -y virt-manager
$ sudo systemctl enable libvirtd
$ sudo systemctl start libvirtd
```
Install win10 using following configurations:    

![/images/2023_02_24_16_15_04_428x466.jpg](/images/2023_02_24_16_15_04_428x466.jpg)

vm configuration:    

![/images/2023_02_24_16_15_25_391x206.jpg](/images/2023_02_24_16_15_25_391x206.jpg)



Added the qemuargs:    

```
virt-xml win10 --edit --confirm --qemu-commandline '-set device.hostpci0.x-igd-gms=1'
```
### libvirt configuration
libvirt hooks for pre-set vfio and post release vfio equipments:    

```
root@idv10:/etc/libvirt/hooks# cat qemu 
#!/bin/bash

OBJECT="$1"
OPERATION="$2"

if [[ $OBJECT == "win10" ]]; then
	case "$OPERATION" in
        	"prepare")
                systemctl start libvirt-nosleep@"$OBJECT"  2>&1 | tee -a /var/log/libvirt/custom_hooks.log
                /bin/vfio-startup.sh 2>&1 | tee -a /var/log/libvirt/custom_hooks.log
                ;;

            "release")
                systemctl stop libvirt-nosleep@"$OBJECT"  2>&1 | tee -a /var/log/libvirt/custom_hooks.log  
                /bin/vfio-teardown.sh 2>&1 | tee -a /var/log/libvirt/custom_hooks.log
                ;;
	esac
fi
root@idv10:/etc/libvirt/hooks# cat vfio-startup.sh 
#!/bin/bash
# Helpful to read output when debugging
set -x

long_delay=10
medium_delay=5
short_delay=1
echo "Beginning of startup!"

function stop_display_manager_if_running {
    # Stop dm using systemd
    if command -v systemctl; then
        if systemctl is-active --quiet "$1.service" ; then
            echo $1 >> /tmp/vfio-store-display-manager
            systemctl stop "$1.service"
        fi

        while systemctl is-active --quiet "$1.service" ; do
            sleep "${medium_delay}"
        done

        return
    fi

    # Stop dm using runit
    if command -v sv; then
        if sv status $1 ; then
            echo $1 >> /tmp/vfio-store-display-manager
            sv stop $1
        fi
    fi
}


# Stop currently running display manager
if test -e "/tmp/vfio-store-display-manager" ; then
    rm -f /tmp/vfio-store-display-manager
fi

stop_display_manager_if_running sddm
stop_display_manager_if_running gdm
stop_display_manager_if_running lightdm
stop_display_manager_if_running lxdm
stop_display_manager_if_running xdm
stop_display_manager_if_running mdm
stop_display_manager_if_running display-manager

sleep "${medium_delay}"

# Unbind VTconsoles if currently bound (adapted from https://www.kernel.org/doc/Documentation/fb/fbcon.txt)
if test -e "/tmp/vfio-bound-consoles" ; then
    rm -f /tmp/vfio-bound-consoles
fi
for (( i = 0; i < 16; i++))
do
  if test -x /sys/class/vtconsole/vtcon${i}; then
      if [ `cat /sys/class/vtconsole/vtcon${i}/name | grep -c "frame buffer"` \
           = 1 ]; then
	       echo 0 > /sys/class/vtconsole/vtcon${i}/bind
           echo "Unbinding console ${i}"
           echo $i >> /tmp/vfio-bound-consoles
      fi
  fi
done

# Unbind EFI-Framebuffer
if test -e "/tmp/vfio-is-nvidia" ; then
    rm -f /tmp/vfio-is-nvidia
fi

if lsmod | grep "nvidia" &> /dev/null ; then
    echo "true" >> /tmp/vfio-is-nvidia
    echo efi-framebuffer.0 > /sys/bus/platform/drivers/efi-framebuffer/unbind
fi

igd_id="8086 $(lspci -n|grep '0:02.0'|cut -d ':' -f4|cut -c 1-4)"
usb_id="8086 $(lspci -n|grep '00:14.0'|cut -d ':' -f4|cut -c 1-4)"
echo 0000:00:02.0 > /sys/bus/pci/drivers/i915/unbind
echo 0000:00:14.0 > /sys/bus/pci/drivers/xhci_hcd/unbind
if ! lsmod | grep "vfio_pci" &> /dev/null ; then
    modprobe vfio-pci
fi
echo $igd_id > /sys/bus/pci/drivers/vfio-pci/new_id
echo $usb_id > /sys/bus/pci/drivers/vfio-pci/new_id

echo "End of startup!"
root@idv10:/etc/libvirt/hooks# cat vfio-teardown.sh 
#!/bin/bash
set -x

# on shutoff state then you could unbind the vfio-pcis 
#sleep 20
#a="fucku"
#until [[ $a == "shut off" ]]
#do
#	a=`virsh domstate win10`
#	sleep 3
#done

ia_addr="0000:$(lspci|grep 'Audio'|grep 'Intel'|cut -c 1-7)"
usb_addr="0000:$(lspci|grep 'USB'|grep 'Intel'|cut -c 1-7)"
igd_id="8086 $(lspci -n|grep '0:02.0'|cut -d ':' -f4|cut -c 1-4)"

echo 0000:00:02.0 > /sys/bus/pci/drivers/vfio-pci/unbind
echo $ia_addr > /sys/bus/pci/drivers/vfio-pci/unbind
echo $usb_addr > /sys/bus/pci/drivers/vfio-pci/unbind
echo $igd_id > /sys/bus/pci/drivers/vfio-pci/remove_id
echo 0000:00:02.0 > /sys/bus/pci/drivers/i915/bind
echo $ia_addr > /sys/bus/pci/drivers/snd_hda_intel/bind
echo $usb_addr >/sys/bus/pci/drivers/xhci_hcd/bind

echo "Beginning of teardown!"

sleep 10

# Restart Display Manager
input="/tmp/vfio-store-display-manager"
while read displayManager; do
  if command -v systemctl; then
    systemctl start "$displayManager.service"
  else
    if command -v sv; then
      sv start $displayManager
    fi
  fi
done < "$input"

# Rebind VT consoles (adapted from https://www.kernel.org/doc/Documentation/fb/fbcon.txt)
input="/tmp/vfio-bound-consoles"
while read consoleNumber; do
  if test -x /sys/class/vtconsole/vtcon${consoleNumber}; then
      if [ `cat /sys/class/vtconsole/vtcon${consoleNumber}/name | grep -c "frame buffer"` \
           = 1 ]; then
    echo "Rebinding console ${consoleNumber}"
	  echo 1 > /sys/class/vtconsole/vtcon${consoleNumber}/bind
      fi
  fi
done < "$input"

# Rebind framebuffer for nvidia
if test -e "/tmp/vfio-is-nvidia" ; then
  echo "efi-framebuffer.0" > /sys/bus/platform/drivers/efi-framebuffer/bind
fi


echo "End of teardown!"

```

### kde reback
kde startup command in desktop, vm exit will send back to desktop.   
