+++
title = "gitlabciandgitbook"
date = "2018-03-27T16:15:46+08:00"
description = "gitlabciandgitbook"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 环境
gitlab搭建完毕，gitlabci配置完毕。    
目的: 整合从代码到发布一条龙服务。    

代码提交->生成中间制品pdf.    
代码提交-> 更新静态网站.    

### 步骤
创建一个新项目:    

![/images/2018_03_27_16_19_37_576x192.jpg](/images/2018_03_27_16_19_37_576x192.jpg)

命名该项目:    

![/images/2018_03_27_16_20_02_605x466.jpg](/images/2018_03_27_16_20_02_605x466.jpg)

将已有的代码(书)提交到代码仓库:    

```
# git init
# git remote add origin http://192.168.109.2/root/mygitbook.git
# git add .
# git commit -m "Initial commit"
# git push -u origin master
```
与此同时，注册runner, 并保证runner所依赖的容器镜像就绪:    

```
[root@csnode1 ~]# gitlab-ci-multi-runner register
Running in system-mode.                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.109.2/
Please enter the gitlab-ci token for this runner:
4dEFxKVNcmnoStBBDssd
Please enter the gitlab-ci description for this runner:
[csnode1]: myrunner
Please enter the gitlab-ci tags for this runner (comma separated):
myrunner tag
Whether to run untagged builds [true/false]:
[false]: 
Whether to lock Runner to current project [true/false]:
[false]: 
Registering runner... succeeded                     runner=4dEFxKVN
Please enter the executor: ssh, kubernetes, docker, docker-ssh, parallels, docker-ssh+machine, shell, virtualbox, docker+machine:
docker
Please enter the default Docker image (e.g. ruby:2.1):
mygitbook:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

runner失败:    

![/images/2018_03_28_14_01_12_468x180.jpg](/images/2018_03_28_14_01_12_468x180.jpg)

配置镜像的拉取规则:    

```
# vim /etc/gitlab-runner/config.toml 
[[runners]]
  name = "myrunner"
  url = "http://192.168.109.2/"
  token = "8996297cd5d5d80bd23c0acade3b95"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "mygitbook:latest"
    privileged = false
    disable_cache = false
    volumes = ["/cache"]
    pull_policy = "if-not-present"
    shm_size = 0
  [runners.cache]
# gitlab-runner restart
```

对应的gitlab定义文件如下:    

```
image: "my_gitbook:latest"
stages:
  - gitbook_build_deploy

gitbook_build_deploy:
  stage: gitbook_build_deploy
  script:
    - gitbook build
    - gitbook pdf
    - ssh-keyscan -H 192.168.109.2 >> ~/.ssh/known_hosts
    - pwd && ls -l -h ./_book  && sshpass -p vagrant scp -P 22 -r _book/* vagrant@192.168.109.2:/home/vagrant/tmp
  artifacts:
    paths:
      - ./book.pdf
  tags:
    - myrunnertag
```
每次build将自动生成pdf供下载。

