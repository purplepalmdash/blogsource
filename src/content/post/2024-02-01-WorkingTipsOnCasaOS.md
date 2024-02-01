+++
title= "WorkingTipsOnCasaOS"
date = "2024-02-01T10:20:04+08:00"
description = "WorkingTipsOnCasaOS"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install:    

```
curl -fsSL https://get.casaos.io | sudo bash
```
Issue:    

```
[FAILED] rclone.service is not running, Please reinstall.
```
Solved via:    

```
cp /bin/mkdir /usr/bin/mkdir
cp /bin/rm /usr/bin/rm
```
Re-run:    

```
curl -fsSL https://get.casaos.io | sudo bash
```

![/images/2024_02_01_10_24_25_515x531.jpg](/images/2024_02_01_10_24_25_515x531.jpg)

Using docker for running redroid:     

```
version: "3"
services:
  redroid:
    image: redroid12:latest
    stdin_open: true
    tty: true
    privileged: true
    ports:
      - "5555:5555"
    volumes:
    # 資料存放在目前目錄下
      - ./redroid-12-data:/data
    command:
    # 禁用GPU硬體加速
      - androidboot.redroid_gpu_mode=guest
      - androidboot.use_memfd=1

  scrcpy:
    image: emptysuns/scrcpy-web:v0.1
    privileged: true
    ports:
      - "48000:8000"
    volumes:
    # 資料存放在目前目錄下
      - ./scrcpy-data:/data
    links:
    # always using myphone1
      - redroid:myphone1
```
Connect via:    

```
docker exec -it redroid_scrcpy_1 adb connect myphone1:5555
```
做到casaos的编排里。  

![/images/2024_02_01_11_24_14_796x805.jpg](/images/2024_02_01_11_24_14_796x805.jpg)

![/images/2024_02_01_11_25_51_799x841.jpg](/images/2024_02_01_11_25_51_799x841.jpg)

![/images/2024_02_01_11_26_41_796x796.jpg](/images/2024_02_01_11_26_41_796x796.jpg)

![/images/2024_02_01_11_39_13_616x435.jpg](/images/2024_02_01_11_39_13_616x435.jpg)
 
