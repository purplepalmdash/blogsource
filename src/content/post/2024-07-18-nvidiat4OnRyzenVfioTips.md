+++
title= "nvidiat4OnRyzenVfioTips"
date = "2024-07-18T17:50:18+08:00"
description = "nvidiat4OnRyzenVfioTips"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Host Preparation
Hardware information:      

```
root@hope:/etc/libvirt# sudo lspci -nn| grep -i nvidia
08:00.0 3D controller [0302]: NVIDIA Corporation TU104GL [Tesla T4] [10de:1eb8] (rev a1)
root@hope:/etc/libvirt# lscpu | grep -i model
Model:                              96
Model name:                         AMD Ryzen 5 4500 6-Core Processor
```
Edit the grub configuration:       

```
$ sudo vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt kvm.ignore_msrs=1 video=efifb:off vfio-pci.ids=10de:1eb8"
$ sudo update-grub2
$ sudo vim /etc/initramfs-tools/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
Specify the vfio driver for nvidia t4:       

```
$ sudo vim /etc/modprobe.d/vfio.conf 
options vfio-pci ids=10de:1eb8
$ sudo update-initramfs -u -k all
```
Download the vbios for nvidia t4 from `https://www.techpowerup.com/vgabios/259926/259926`.     

After reboot, check the driver status:      

```
dash@hope:~$ lspci -vvnn -s 08:00.0
08:00.0 3D controller [0302]: NVIDIA Corporation TU104GL [Tesla T4] [10de:1eb8] (rev a1)
	Subsystem: NVIDIA Corporation TU104GL [Tesla T4] [10de:12a2]
	Control: I/O- Mem- BusMaster- SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Interrupt: pin A routed to IRQ 255
	Region 0: Memory at fb000000 (32-bit, non-prefetchable) [disabled] [size=16M]
	Region 1: Memory at ffc0000000 (64-bit, prefetchable) [disabled] [size=256M]
	Region 3: Memory at fff0000000 (64-bit, prefetchable) [disabled] [size=32M]
	Capabilities: <access denied>
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau
```
### vm setup
UEFI setting:    

![/images/20240718_180958_x.jpg](/images/20240718_180958_x.jpg)

Continue for installation, until it finished.   


```
sudo apt install -y libevent-dev build-essential vim
sudo apt-get upgrade
sudo shutdown -h now
```
Shutdown and add the tesla t4:    

![/images/20240718_185046_x.jpg](/images/20240718_185046_x.jpg)

Change the video to none:    

![/images/20240718_185215_x.jpg](/images/20240718_185215_x.jpg)

Start, and from now on, you could only ssh into the machine.   

### nvidia driver installation
Steps are listed as following:      

```
distro=ubuntu2204
arch=x86_64
wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb 
sudo apt-get install cuda-toolkit
sudo apt-get install nvidia-gds
sudo ubuntu-drivers autoinstall
sudo apt-get install --install-recommends linux-generic-hwe-22.04
```
Only in hwe kernel, nvidia-smi could be running properly.    

```
$ vim ~/.bashrc
# cuda related
export PATH=/usr/local/cuda-12.5/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-12.5/lib64\
                         ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
Examine the nvcc version:       

```
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on Thu_Jun__6_02:18:23_PDT_2024
Cuda compilation tools, release 12.5, V12.5.82
Build cuda_12.5.r12.5/compiler.34385749_0
```
Examine the card info:      

```
$ sudo nvidia-smi 
Thu Jul 18 12:09:39 2024       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 555.42.06              Driver Version: 555.42.06      CUDA Version: 12.5     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  Tesla T4                       Off |   00000000:07:00.0 Off |                    0 |
| N/A   35C    P8              9W /   70W |       1MiB /  15360MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```
### ComfyUI Setup
Install git-lfs:      

```
$ sudo apt install -y git git-lfs
$ git lfs install
```
Install miniconda:     

```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash
```
Relogin the terminal.     

Install ComfyUI:    

```
$ cd Code
$ git clone https://github.com/comfyanonymous/ComfyUI.git
$ conda create -n comfyui python=3.10
$ pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
$ pip3 install torch torchvision torchaudio
```
Install :     

```
git clone https://github.com/Limitex/ComfyUI-Diffusers.git
cd ComfyUI-Diffusers
pip install -r requirements.txt
git clone https://github.com/cumulo-autumn/StreamDiffusion.git
python -m streamdiffusion.tools.install-tensorrt
```
Configure:      

```
$ sudo apt-get install -y nfs-common
$ sudo mkdir -p /media/nfs
$ sudo mount model_on_nfs /media/nfs
$ cd ~/Code/ComfyUI
$ cp extra_model_paths.yaml.example extra_model_paths.yaml
$ vim extra_model_paths.yaml
a111: 
    base_path: /media/nfs/stable-diffusion-webui/
goto ComfyUI/custom_nodes dir in terminal(cmd)
$ git clone https://github.com/ltdrdata/ComfyUI-Manager.git
Restart ComfyUI
$ python main.py --port 8188 --listen 192.168.1.60
```
