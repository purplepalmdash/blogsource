+++
title= "Changemysimplescreenrecoder"
date = "2024-12-09T09:11:09+08:00"
description = "Changemysimplescreenrecoder"
keywords = ["Technology"]
categories = ["Technology"]
+++
Change the command:    

```
    #### 7. mysimplerecorder
    (writeShellScriptBin "mysimplerecorder" ''
    rm -f /tmp/tmprecording.mp4
    wf-recorder -g "$(slurp)" -f /tmp/tmprecording.mp4 -c h264_vaapi -d /dev/dri/renderD128 -p "preset=superfast"
    cp /tmp/tmprecording.mp4 /home/dash/Videos/`date +"%Y-%m-%d-%H-%M-%S" `.mp4
  '')

```
Since we want to use `h264_vaapi`, we need to enable vaapi under nixos's configuration:     

```
  nixpkgs.config.packageOverrides = pkgs: {
    intel-vaapi-driver = pkgs.intel-vaapi-driver.override { enableHybridCodec = true; };
  };
  hardware.graphics = { # hardware.graphics since NixOS 24.11
    enable = true;
    extraPackages = with pkgs; [
      intel-media-driver # LIBVA_DRIVER_NAME=iHD
      intel-vaapi-driver # LIBVA_DRIVER_NAME=i965 (older but works better for Firefox/Chromium)
      libvdpau-va-gl
    ];
  };
  environment.sessionVariables = { LIBVA_DRIVER_NAME = "iHD"; }; # Force intel-media-driver
```
Rebuild then you could use new comand for screen recorder:     

```
sudo nixos-rebuild switch --option substituers https://mirror.sjtu.edu.cn/nix-channels/store
```
