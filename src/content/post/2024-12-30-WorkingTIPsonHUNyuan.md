+++
title= "WorkingTIPsonHUNyuan"
date = "2024-12-30T09:14:52+08:00"
description = "WorkingTIPsonHUNyuan"
keywords = ["Technology"]
categories = ["Technology"]
+++
### update comfyui
update comfyui via comfyUI manager, then you got error:    

```
Error. No naistyles.csv found. Put your naistyles.csv in the custom_nodes/ComfyUI_NAI-mod/CSV directory of ComfyUI. Then press "Refresh".
                  Your current root directory is: /home/dash/Code/ComfyUI
            

```
Solved via:    

```
Create a ComfyUI-NAI-styler directory under the custom_nodes directory.
Create an __init__.py file under the ComfyUI-NAI-styler directory with the following content:
__all__ = ['NODE_CLASS_MAPPINGS', 'NODE_DISPLAY_NAME_MAPPINGS']
NODE_CLASS_MAPPINGS = {}
NODE_DISPLAY_NAME_MAPPINGS = {}
Create a CSV directory under the ComfyUI-NAI-styler directory .

Create three empty files under the CSV directory:

naifilters.csv
naistyles.csv
naitypes.csv


(base) dash@comfyvm:~/Code/ComfyUI/custom_nodes$ cp -r ComfyUI-NAI-styler ComfyUI_NAI-mod

(base) dash@comfyvm:~/Code/ComfyUI/custom_nodes$ pwd
/home/dash/Code/ComfyUI/custom_nodes
(base) dash@comfyvm:~/Code/ComfyUI/custom_nodes$ vim ComfyUI-Universal-Styler/naistyler_nodes.py 

    def INPUT_TYPES(cls):
        base_path = Path(folder_paths.base_path) # Use of Path to ensure path compatibility
        cls.naistyles_csv = cls.load_naistyles_csv(base_path / "custom_nodes/ComfyUI-Universal-Styler/CSV/naistyles.csv")
        cls.naifilters_csv = cls.load_naifilters_csv(base_path / "custom_nodes/ComfyUI-Universal-Styler/CSV/naifilters.csv")
        cls.naitypes_csv = cls.load_naitypes_csv(base_path / "custom_nodes/ComfyUI-Universal-Styler/CSV/naitypes.csv")
        #cls.naistyles_csv = cls.load_naistyles_csv(os.path.join(folder_paths.base_path, "custom_nodes\\ComfyUI-NAI-styler\\CSV\\naifilters.csv"))
        #cls.naifilters_csv = cls.load_naifilters_csv(os.path.join(folder_paths.base_path, "custom_nodes\\ComfyUI-NAI-styler\\CSV\\naistyles.csv"))
        #cls.naitypes_csv = cls.load_naitypes_csv(os.path.join(folder_paths.base_path, "custom_nodes\\ComfyUI-NAI-Styler\\CSV\\naitypes.csv"))

```
### comfyui
Install missing custom nodes:    

![/images/2024_12_30_09_33_23_1300x716.jpg](/images/2024_12_30_09_33_23_1300x716.jpg)

After restarting, the comfyui will be shown like:    

![/images/2024_12_30_09_34_09_609x369.jpg](/images/2024_12_30_09_34_09_609x369.jpg)

models you should download:     

```
https://hf-mirror.com/city96/HunyuanVideo-gguf/tree/main
hunyuan-video-t2v-720p-Q5_0.gguf

https://hf-mirror.com/Comfy-Org/HunyuanVideo_repackaged/blob/main/split_files/text_encoders/llava_llama3_fp8_scaled.safetensors

https://hf-mirror.com/Kijai/HunyuanVideo_comfy/blob/main/hunyuan_video_vae_bf16.safetensors

```
### comfyui II
Download models:     

```
https://hf-mirror.com/Kijai/HunyuanVideo_comfy/blob/main/hunyuan_video_720_cfgdistill_fp8_e4m3fn.safetensors
https://hf-mirror.com/Kijai/HunyuanVideo_comfy/blob/main/hunyuan_video_vae_bf16.safetensors

```
clone the llama:   

```
(comfyui) dash@comfyvm:~/Code/ComfyUI$ cd models/LLM/
(comfyui) dash@comfyvm:~/Code/ComfyUI/models/LLM$ 
git clone https://hf-mirror.com/Kijai/llava-llama-3-8b-text-encoder-tokenizer
```
clone the clip items:    

```
(base) dash@comfyvm:~$ cd ~/Code/ComfyUI/models/clip
(base) dash@comfyvm:~/Code/ComfyUI/models/clip$ ls
clip_l.safetensors  llava_llama3_fp8_scaled.safetensors  put_clip_or_text_encoder_models_here
(base) dash@comfyvm:~/Code/ComfyUI/models/clip$ 

git clone https://hf-mirror.com/openai/clip-vit-large-patch14
```
