+++
title= "WorkingTipsOnOpenSora"
date = "2024-03-26T15:33:55+08:00"
description = "WorkingTipsOnOpenSora"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Environment
Ubuntu22.04, with vfio pass-through nvidia card.     

```
$ sudo apt update -y && sudo apt upgrade -y
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
Install cuda(12.1.1-1):     

```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
sudo apt install cuda=12.1.1-1
```
Activate the conda environment:    

```
conda create -n opensora python=3.10
(base) dash@vfio2:~$ conda activate opensora
(opensora) dash@vfio2
pip3 install torch torchvision
pip install packaging ninja
pip install flash-attn --no-build-isolation
pip install -v --disable-pip-version-check --no-cache-dir --no-build-isolation --config-settings "--build-option=--cpp_ext" --config-settings "--build-option=--cuda_ext" git+https://github.com/NVIDIA/apex.git
pip3 install -U xformers --index-url https://download.pytorch.org/whl/cu121
git clone https://github.com/hpcaitech/Open-Sora
cd Open-Sora
pip install -v .
pip install gradio
```
Issue:    

```
https://github.com/hpcaitech/Open-Sora/issues/209
```
