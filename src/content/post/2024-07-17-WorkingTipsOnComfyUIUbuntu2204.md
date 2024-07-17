+++
title= "WorkingTipsOnComfyUIUbuntu2204"
date = "2024-07-17T11:07:48+08:00"
description = "WorkingTipsOnComfyUIUbuntu2204"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Installation
Ubuntu22.04, with a6000, install steps:      

```
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
conda create -n comfyui python=3.10
conda activate comfyui
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install torch torchvision torchaudio
pip install -r requirements.txt 
 cp extra_model_paths.yaml.example extra_model_paths.yaml
 vim extra_model_paths.yaml
 cd models/
 ls
 cd ..
 vim extra_model_paths.yaml
 python main.py --port 8188 --listen 192.168.1.7
```
Install :      

```
sudo apt install nvidia-cudnn
```
Install ComfyUI manager:    

```
goto ComfyUI/custom_nodes dir in terminal(cmd)
git clone https://github.com/ltdrdata/ComfyUI-Manager.git
Restart ComfyUI
```
