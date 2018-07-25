+++
title = "MonocularChinese"
date = "2018-07-25T08:15:32+08:00"
description = "MonocularChinese"
keywords = ["Linux"]
categories = ["Linux"]
+++
Moncular汉化过程，记录如下:    

```
# git clone https://github.com/kubernetes-helm/monocular.git
# cd monocular
# vim dev_env/ui/Dockerfile
    FROM bitnami/node:8
    
    # Install yarn
    RUN install_packages gnupg2
    RUN install_packages apt-transport-https && \
      curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
      #echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
      echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list 
    RUN install_packages yarn
```
全程需要翻墙。   
其中需要下载一大堆镜像，运行成功后用浏览器访问`http://localhost:4200`即可。在目录下编辑对应的文件，更改中文即可看到效果:    

![/images/2018_07_25_09_10_33_735x531.jpg](/images/2018_07_25_09_10_33_735x531.jpg)

