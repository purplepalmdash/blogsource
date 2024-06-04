+++
title= "OnGentooInstallation"
date = "2024-06-03T09:08:37+08:00"
description = "OnGentooInstallation"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. Preparation
Start sshd:    

```
/etc/init.d/sshd start
passwd root
```
Remove the lvm via:    

```
# dmsetup ls
# dmsetup remove xxxxxx
```
lsblk:    

```
# lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0 479.7M  1 loop /mnt/livecd
sda           8:0    0   1.8T  0 disk 
sr0          11:0    1  1024M  0 rom  
sr1          11:1    1 527.4M  0 rom  /mnt/cdrom
zram0       253:0    0     0B  0 disk 
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0   512M  0 part 
├─nvme0n1p2 259:2    0     1G  0 part 
└─nvme0n1p3 259:3    0 475.4G  0 part 
```
Partition:     

```
livecd ~ # mkfs.vfat -F32 /dev/nvme0n1p1
mkfs.fat 4.2 (2021-01-31)
livecd ~ # mkswap /dev/nvme0n1p2
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=5d36c31a-6fbf-4d04-84ac-9ee1efaf5712
livecd ~ # mkfs.ext4 /dev/nvme0n1p3
livecd ~ # swapon /dev/nvme0n1p2 
livecd ~ # mount /dev/nvme0n1p3 /mnt/gentoo
```
Get the stage3 file and untar it:    

```
wget https://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/20240602T164858Z/stage3-amd64-desktop-openrc-20240602T164858Z.tar.xz
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```
enable makeopts:     

```
# nano /etc/portage/make.conf
......
MAKEOPTS="-j20"
......
```
Select mirror:     

```
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
... select ustc.edu.cn
```

Copy the Portage repository settings:     

```
livecd /mnt/gentoo # mkdir -p /mnt/gentoo/etc/portage/repos.conf
livecd /mnt/gentoo # cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```
Copy the dns:     

```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```
Chroot to system:     

```
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

Mount EFI partition:      

```
livecd / # mount /dev/nvme0n1p1 /boot
```
Update Gentoo's ebuild :     

```
emerge-webrsync
```
Update using portage:      

```
emerge --ask --verbose --update --deep --newuse @world
```
Update to `Asia/Shanghai`:     

```
livecd / # echo "Asia/Shanghai">/etc/timezone
livecd / # emerge --config sys-libs/timezone-data
```
Edit the locale.gen:     

```
# nano /etc/locale.gen
en_US.UTF-8 UTF-8
# locale-gen
```
Reload the setting:      

```
env-update
source /etc/profile
export PS1="(chroot) ${PS1}"
```

### 2. Kernel
Edit cpu flags:      

```
#  emerge --ask app-portage/cpuid2cpuflags
#  cpuid2cpuflags
CPU_FLAGS_X86: aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3
# nano /etc/portage/make.conf
CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3"
```

Edit `/etc/portage/make.conf`, accept all of the license tips:      

```
ACCEPT_LICENSE="*"
```
Install firmware and intel microcode:     

```
emerge --ask sys-kernel/linux-firmware  sys-firmware/intel-microcode
```
Change default editor from nano to vim:     

```
emerge -vj app-editors/vim
eselect editor list
eselect editor set 2
. /etc/profile
PS1=(chroot)$PS1
```
Install firmware/kernel/bootloader:     

```
mkdir -p /etc/portage/package.license
echo 'sys-kernel/linux-firmware linux-fw-redistributable no-source-code' >/etc/portage/package.license/linux-firmware
# also install initramfs
echo 'sys-kernel/installkernel dracut' >/etc/portage/package.use/installkernel
emerge -vj linux-firmware gentoo-kernel-bin grub
```
Edit `/etc/fstab`:     

```
# efi partition
#/dev/nvme0n1p1: UUID="846A-BE3E" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="2e1058ce-0964-4624-be29-1ae87beae5fc"
UUID="846A-BE3E"         /boot/efi   vfat  rw,noatime,errors=remount-ro 0 2
# swap partition
#/dev/nvme0n1p2: UUID="5d36c31a-6fbf-4d04-84ac-9ee1efaf5712" TYPE="swap" PARTUUID="a35a309d-e60d-4b4e-8ef4-296962fda56b"
UUID="5d36c31a-6fbf-4d04-84ac-9ee1efaf5712"	none	swap  sw                           0 0
# root partition
#/dev/nvme0n1p3: UUID="2c24a130-dc28-40fd-ad92-2fd68c75229e" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="43320d2d-f849-4b45-a14c-7ac254b330df"
UUID="2c24a130-dc28-40fd-ad92-2fd68c75229e"  /           ext4  defaults,noatime             0 1
```
Install UEFI:    

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi/ --bootloader-id=Gentoo
# notice swap file
sed -Ei "/GRUB_CMDLINE_LINUX_DEFAULT/s/^#*(GRUB.*DEFAULT=).*$/\1\"resume=UUID=$(blkid -o value /dev/nvme0n1p2 | head -1)\"/" /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg
```
os-probe related:    

```
echo 'sys-boot/grub mount' >/etc/portage/package.use/grub
emerge -vj os-prober
echo 'GRUB_DISABLE_OS_PROBER=false' >>/etc/default/grub
```

### Configuration
Create user:    

```
passwd root
useradd -m -G usb,wheel kkk
passwd kkk
emerge -vj app-admin/sudo
visudo    # uncomment #%wheel
```
Install NetworkManager:    

```
echo "net-wireless/wpa_supplicant dbus" >>/etc/portage/package.use/nm
echo "net-misc/openssh -bindist" >>/etc/portage/package.use/nm
emerge -vj1 net-misc/openssh net-misc/networkmanager
emerge -On net-misc/networkmanager
rc-update add NetworkManager default
rc-update add sshd default
```
Install log service:      

```
emerge -vj app-admin/syslog-ng
rc-update add syslog-ng default
```
Reboot:     

```
sync
exit
umount -Rl /mnt/gentoo/{dev,proc,sys,}
reboot
```

### repository
via:    

```
emerge --ask app-eselect/eselect-repository
eselect repository list
eselect repository enable  zugaina
emaint sync -r zugaina

```

### usb key related
Format via:    

```
livecd ~ # lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0 479.7M  1 loop /mnt/livecd
sda           8:0    1  57.3G  0 disk 
├─sda1        8:1    1   1.3G  0 part 
├─sda2        8:2    1   512K  0 part 
└─sda3        8:3    1    56G  0 part 
sr0          11:0    1 527.4M  0 rom  /mnt/cdrom
zram0       253:0    0     0B  0 disk 
nvme0n1     259:0    0 953.9G  0 disk 
├─nvme0n1p1 259:1    0   480M  0 part 
└─nvme0n1p2 259:2    0 953.4G  0 part 
livecd ~ # parted -a optimal /dev/sda
GNU Parted 3.6
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt                                                      
Warning: The existing disk label on /dev/sda will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? yes                                                               
(parted) mkpart primary fat32 0% 100%                                     
(parted) set 1 BOOT on                                                    
(parted) quit                                                             
Information: You may need to update /etc/fstab.

```
Create the gpg:    

```
livecd ~ # export GPG_TTY=$(tty)
livecd ~ # echo $(tty)
/dev/pts/0
livecd ~ # dd if=/dev/urandom bs=8388607 count=1 | gpg --symmetric --cipher-algo AES256 --output /tmp/efiboot/luks-key.gpg
gpg: directory '/root/.gnupg' created
1+0 records in
1+0 records out
8388607 bytes (8.4 MB, 8.0 MiB) copied, 6.76511 s, 1.2 MB/s

```

nvme ssd:    

```
# parted -a optimal /dev/nvme0n1
GNU Parted 3.6
Using /dev/nvme0n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) unit s                                                           
(parted) print free                                                       
(parted) mkpart primary 2048s 2000409230s                                   
(parted) quit                                                             
Information: You may need to update /etc/fstab.
```
Create encrypted disk:    

```
# gpg --decrypt /tmp/efiboot/luks-key.gpg | cryptsetup --cipher serpent-xts-plain64 --key-size 512 --hash whirlpool --key-file - luksFormat /dev/nvme0n1p1 
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
livecd ~ # cryptsetup luksDump /dev/nvme0n1p1

```
disk layout:     

```
livecd ~ # gpg --decrypt /tmp/efiboot/luks-key.gpg | cryptsetup --key-file - luksOpen  /dev/nvme0n1p1 gentoo
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
livecd ~ # ls /dev/mapper/
control  gentoo
livecd ~ # pvcreate /dev/mapper/gentoo
  Physical volume "/dev/mapper/gentoo" successfully created.
livecd ~ # vgcreate vg1 /dev/mapper/gentoo
  Volume group "vg1" successfully created
livecd ~ # grep MemTotal /proc/meminfo
MemTotal:       15954144 kB
livecd ~ # lvcreate --size 8G --name swap vg1
  Logical volume "swap" created.
livecd ~ # lvcreate --size 80G --name root vg1
  Logical volume "root" created.
livecd ~ # lvcreate --extents 95%FREE --name home vg1
  Logical volume "home" created.
livecd ~ # pvdisplay 
  --- Physical volume ---
  PV Name               /dev/mapper/gentoo
  VG Name               vg1
  PV Size               953.85 GiB / not usable <1.32 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              244186
  Free PE               11083
  Allocated PE          233103
  PV UUID               xBgj1j-1e3g-iaT0-esba-jJ3f-EO3m-Ty6wgY
   
livecd ~ # vgdisplay 
  --- Volume group ---
  VG Name               vg1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               953.85 GiB
  PE Size               4.00 MiB
  Total PE              244186
  Alloc PE / Size       233103 / <910.56 GiB
  Free  PE / Size       11083 / 43.29 GiB
  VG UUID               YUdAbE-MnJA-54GY-aQkb-Kp9G-P52q-Wl3Uey
   
livecd ~ # lvdisplay 
  --- Logical volume ---
  LV Path                /dev/vg1/swap
  LV Name                swap
  VG Name                vg1
  LV UUID                OKxWa1-NBN2-F6zx-E1qQ-wPQJ-2lmP-7abjG1
  LV Write Access        read/write
  LV Creation host, time livecd, 2024-06-03 12:41:29 +0000
  LV Status              available
  # open                 0
  LV Size                8.00 GiB
  Current LE             2048
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
   
  --- Logical volume ---
  LV Path                /dev/vg1/root
  LV Name                root
  VG Name                vg1
  LV UUID                yAMQV4-sNI7-zuY8-ZkDX-k53i-dxoF-PsvE4O
  LV Write Access        read/write
  LV Creation host, time livecd, 2024-06-03 12:41:43 +0000
  LV Status              available
  # open                 0
  LV Size                80.00 GiB
  Current LE             20480
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:2
   
  --- Logical volume ---
  LV Path                /dev/vg1/home
  LV Name                home
  VG Name                vg1
  LV UUID                APZd1G-qGe1-9t5c-AhBh-pUKt-Dit8-D2WTHd
  LV Write Access        read/write
  LV Creation host, time livecd, 2024-06-03 12:41:48 +0000
  LV Status              available
  # open                 0
  LV Size                <822.56 GiB
  Current LE             210575
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:3
livecd ~ # vgchange --available y
  3 logical volume(s) in volume group "vg1" now active
livecd ~ # ls /dev/mapper
control  gentoo  vg1-home  vg1-root  vg1-swap
livecd ~ # vgchange --available y
  3 logical volume(s) in volume group "vg1" now active
livecd ~ # ls /dev/mapper
control  gentoo  vg1-home  vg1-root  vg1-swap
livecd ~ # mkswap -L "swap" /dev/mapper/vg1-swap
Setting up swapspace version 1, size = 8 GiB (8589930496 bytes)
LABEL=swap, UUID=b8653482-926c-4e26-89fb-55f52f3fca9f
livecd ~ # mkfs.ext4 -L "root" /dev/mapper/vg1-root
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 20971520 4k blocks and 5242880 inodes
Filesystem UUID: 030ae853-ad94-4fe5-a5a8-340aaeb54a6c
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done   

livecd ~ # mkfs.ext4 -m 0 -L "home" /dev/mapper/vg1-home
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 215628800 4k blocks and 53911552 inodes
Filesystem UUID: f8534406-cdcb-4538-8397-2480d96a306b
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
	102400000, 214990848

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done     

livecd ~ # swapon -v /dev/mapper/vg1-swap
swapon: /dev/mapper/vg1-swap: found signature [pagesize=4096, signature=swap]
swapon: /dev/mapper/vg1-swap: pagesize=4096, swapsize=8589934592, devsize=8589934592
swapon /dev/mapper/vg1-swap
livecd ~ # mount -v -t ext4 /dev/mapper/vg1-root /mnt/gentoo
mount: /dev/mapper/vg1-root mounted on /mnt/gentoo.
livecd ~ # blkid /dev/sda1 /dev/nvme0n1p1 
/dev/sda1: UUID="81A6-BFB2" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="primary" PARTUUID="be83585f-0239-4b8a-8931-674aaaadb69d"
/dev/nvme0n1p1: UUID="ddc06372-5d72-49a0-b617-b97128f8e3e6" TYPE="crypto_LUKS" PARTLABEL="primary" PARTUUID="3635503c-78f0-44ce-8982-633ca20723ff"

```
Receive the gpg key:    

```
gpg --keyserver  pgp.mit.edu --recv-key 2D182910
```
fetch the stage 3 file:    

```
# cd /mnt/gentoo
# wget https://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/20240602T164858Z/stage3-amd64-desktop-openrc-20240602T164858Z.tar.xz
```
untar the stage3 file:     

```
tar xvJpf stage3-amd64-*.tar.xz --xattrs-include='*.*' --numeric-owner
rm -f stage3-amd64-desktop-openrc-20240602T164858Z.tar.xz
```
Edit the bashrc:    

```
livecd /mnt/gentoo # cat /mnt/gentoo/root/.bashrc 
export NUMCPUS=$(nproc)
export NUMCPUSPLUSONE=$(( NUMCPUS + 1 ))
export MAKEOPTS="-j${NUMCPUSPLUSONE} -l${NUMCPUS}"
export EMERGE_DEFAULT_OPTS="--jobs=${NUMCPUSPLUSONE} --load-average=${NUMCPUS}"
```
copy the `bashrc_profile`:     

```
cp -v /mnt/gentoo/etc/skel/.bash_profile /mnt/gentoo/root/
```
edit the make.conf:    

```
livecd /mnt/gentoo # cat /mnt/gentoo/etc/portage/make.conf 
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# detailed example.
COMMON_FLAGS="-march=native -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"

# NOTE: This stage was built with the bindist Use flag enabled

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
#LC_MESSAGES=C.utf8

# Note: MAKEOPTS and EMERGE_DEFAULT_OPTS are set in .bashrc

# The following licence is required, in addition to @FREE, for GNOME.
ACCEPT_LICENSE="CC-Sampling-Plus-1.0"

# WARNING: Changing your CHOST is not something that should be done lightly.
# Please consult http://www.gentoo.org/doc/en/change-chost.xml before changing.
CHOST="x86_64-pc-linux-gnu"

# Use the 'stable' branch - 'testing' no longer required for Gnome 3.
# NB, amd64 is correct for both Intel and AMD 64-bit CPUs
ACCEPT_KEYWORDS="amd64"

# Additional USE flags supplementary to those specified by the current profile.
USE=""
CPU_FLAGS_X86="mmx mmxext sse sse2"

# Important Portage directories.
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C

# Turn on logging - see http://gentoo-en.vfose.ru/wiki/Gentoo_maintenance.
PORTAGE_ELOG_CLASSES="info warn error log qa"
# Echo messages after emerge, also save to /var/log/portage/elog
PORTAGE_ELOG_SYSTEM="echo save"

# Ensure elogs saved in category subdirectories.
# Build binary packages as a byproduct of each emerge, a useful backup.
FEATURES="split-elog buildpkg"

# Settings for X11
VIDEO_CARDS="intel i965"
INPUT_DEVICES="libinput"
```
Select ustc for the mirror, thus  make.conf should be like following:    

```
livecd /mnt/gentoo # tail /mnt/gentoo/etc/portage/make.conf 
# Ensure elogs saved in category subdirectories.
# Build binary packages as a byproduct of each emerge, a useful backup.
FEATURES="split-elog buildpkg"

# Settings for X11
VIDEO_CARDS="intel i965"
INPUT_DEVICES="libinput"

GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/ \
    rsync://rsync.mirrors.ustc.edu.cn/gentoo/"

```

Edit the `repos.conf/gentoo.conf`:      

```
livecd /mnt/gentoo # cp -v /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf


livecd /mnt/gentoo # cat /mnt/gentoo/etc/portage/repos.conf/gentoo.conf 
[DEFAULT]
main-repo = gentoo

[gentoo]
location = /var/db/repos/gentoo
sync-type = webrsync
sync-uri = rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage
auto-sync = yes
sync-rsync-verify-jobs = 1
sync-rsync-verify-metamanifest = yes
sync-rsync-verify-max-age = 3
sync-openpgp-key-path = /usr/share/openpgp-keys/gentoo-release.asc
sync-openpgp-keyserver = hkps://keys.gentoo.org
sync-openpgp-key-refresh-retry-count = 40
sync-openpgp-key-refresh-retry-overall-timeout = 1200
sync-openpgp-key-refresh-retry-delay-exp-base = 2
sync-openpgp-key-refresh-retry-delay-max = 60
sync-openpgp-key-refresh-retry-delay-mult = 4
sync-webrsync-verify-signature = yes

```
Edit the dns file:     

```
# cat /mnt/gentoo/etc/resolv.conf 
nameserver 223.5.5.5
```
Prepare for chroot:     

```
livecd /mnt/gentoo # mount -v -t proc none /mnt/gentoo/proc
mount: none mounted on /mnt/gentoo/proc.
livecd /mnt/gentoo # mount -v --rbind /sys /mnt/gentoo/sys
mount: /sys bound on /mnt/gentoo/sys.
livecd /mnt/gentoo # mount -v --rbind /dev /mnt/gentoo/dev
mount: /dev bound on /mnt/gentoo/dev.
livecd /mnt/gentoo # mount -v --make-rslave /mnt/gentoo/sys
livecd /mnt/gentoo # mount -v --make-rslave /mnt/gentoo/dev
```
chroot in:     

```
livecd /mnt/gentoo # chroot /mnt/gentoo /bin/bash
livecd / # source /etc/profile
livecd / # export PS1="(chroot) $PS1
```
Sync and select profile:     

```
 emaint sync --auto
 eselect profile list
 eselect profile set "default/linux/amd64/17.1"
 emerge --info
 grep -i useflag /var/db/repos/gentoo/profiles/use.desc
 emerge --ask --verbose --oneshot portage

```
set timezone and locale:      

```
 echo "Asia/Shanghai">/etc/timezone
 emerge -v --config sys-libs/timezone-data
 nano -w /etc/locale.gen
 locale-gen
 eselect locale list
 eselect locale set "C"
 env-update && source /etc/profile && export PS1="(chroot) $PS1"
```

set cpu flags in make.conf:      

```
emerge --verbose --oneshot app-portage/cpuid2cpuflags
cpuid2cpuflags
nano -w /etc/portage/make.conf
```

Update using portage:      

```
emerge --ask --verbose --update --deep --newuse @world
```
build kernel:     

```
(chroot) livecd / # mkdir -p -v /etc/portage/package.license
mkdir: created directory '/etc/portage/package.license'
(chroot) livecd / # touch /etc/portage/package.license/zzz_via_autounmas
echo "sys-kernel/linux-firmware linux-fw-redistributable no-source-code" >> /etc/portage/package.license/linux-firmware
emerge --ask --verbose sys-kernel/gentoo-sources
emerge --ask --verbose sys-kernel/linux-firmware
(chroot) livecd / # readlink -v /usr/src/linux
readlink: /usr/src/linux: No such file or directory
(chroot) livecd / # eselect kernel list
Available kernel symlink targets:
  [1]   linux-6.6.30-gentoo
(chroot) livecd / # eselect kernel set 1
(chroot) livecd / # readlink -v /usr/src/linux
linux-6.6.30-gentoo
emerge --ask --verbose dev-vcs/git 
 # cat /etc/portage/repos.conf/waffle-builds.conf
[waffle-builds]
 
# Various utility ebuilds for Gentoo on EFI
# Maintainer: sakaki (sakaki@deciban.com)
 
location = /usr/local/portage/waffle-builds
sync-type = git
sync-uri = https://github.com/FlyingWaffleDev/waffle-builds.git
priority = 50
auto-sync = yes
(chroot) livecd / # emaint sync --repo waffle-builds
(chroot) livecd / # echo "*/*::waffle-builds ~amd64">> /etc/portage/package.accept_keywords/waffle-builds-repo

```
