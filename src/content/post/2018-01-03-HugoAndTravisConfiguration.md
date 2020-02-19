+++
title = "HugoAndTravisConfiguration"
date = "2018-01-03T10:47:18+08:00"
description = "HugoAndTravisConfiguration"
keywords = ["DevOps"]
categories = ["Technology"]
+++
hugo
升级到v0.33后，生成的静态网站里缺少了index.html等HTML文件，原因不明。恰好我在travis上的编译流程有些繁琐，每次都需要花费5分钟以上生成整个网站，于是我调研了一下更好的解决方案，用于生成博客静态网站。    

Issue:    

![/images/2018_01_03_11_14_02_828x601.jpg](/images/2018_01_03_11_14_02_828x601.jpg)

无法正常显示的网页:    

![/images/2018_01_03_11_14_25_829x531.jpg](/images/2018_01_03_11_14_25_829x531.jpg)

### 准备
以前的hugo源代码我直接放在某个仓库里，建了两个分支source和master，
source用于存放源代码，master则是编译后的静态网站结果，编译完成以后，直接发布在github
page上。    

每一次编译都需要推送代码到travis上，在travis的容器里安装好hugo，
安装hugo的方式是直接`go get -u -v`,
这样相当于每次编译网站都需要预先编译出来一个hugo，因而比较耗时。    

为了避免这种繁琐的过程，我建立一个新的仓库，blogsource，将以前的仓库清空,
重新组织目录结构.    

在github上建立一个token, 注意选中其repo选项:    

![/images/2018_01_03_11_14_38_674x523.jpg](/images/2018_01_03_11_14_38_674x523.jpg)

得到的token如下，我们将复制这个数值，后面在travisCI中需要填到:    

![/images/2018_01_03_11_14_48_831x372.jpg](/images/2018_01_03_11_14_48_831x372.jpg)

在travisci的环境变量中，手动添加我们刚才生成的token:    

![/images/2018_01_03_11_16_06_1247x346.jpg](/images/2018_01_03_11_16_06_1247x346.jpg)

### 代码重架构
目录架构如下：    

![/images/2018_01_03_11_17_20_681x240.jpg](/images/2018_01_03_11_17_20_681x240.jpg)

其中binaries下是我们手动下载的hugo可执行文件，下载地址在:     

[https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases)    

我这里选择的是v31.1的Linux-64.tar.gz, 下载完毕后，直接解压到此目录下即可。    

hugo原有的配置文件不需要变化，所需要变化的是我们的.travis.yaml文件。    

### travis配置文件
简单起见，下面直接贴出源代码。    

```
addons:
  apt:
    packages:
      - python-pygments

install:
  - rm -rf public || exit 0

script:
  - cd src
  - ../binaries/hugo --theme=hyde-a

deploy:
  provider: pages
  skip_cleanup: true
  local_dir: public
  github_token: $GITHUB_TOKEN # Set in travis-ci.org dashboard
  on:
    branch: master
  repo: purplepalmdash/purplepalmdash.github.io
  target_branch: master
```

注意到上面的deploy环节，使用到了我们上面生成的token,
而每次编译完成以后，直接推送到`purplepalmdash/purplepalmdash.github.io`仓库的master分支，按照github的说明，这个分支直接提供了博客网站。    

### 更改一些源文件
现在github
page已经完全支持https了，而之前我们的网站使用的是http方式，为此，我们的css文件配置需要有以下的修改，主要是在我的模板文件中定义:    

![/images/2018_01_03_11_22_13_1072x880.jpg](/images/2018_01_03_11_22_13_1072x880.jpg)

更改完毕以后，每次就可以使用https来访问网站了。   

### travisCI自编译
去掉以前的travisCI配置，更新一个新的配置.    

![/images/2018_01_03_11_23_37_482x392.jpg](/images/2018_01_03_11_23_37_482x392.jpg)

完整的配置应该如下:    

![/images/2018_01_03_11_24_02_847x734.jpg](/images/2018_01_03_11_24_02_847x734.jpg)

现在每次针对代码库blogsource的提交，将自动触发博客的编译，并自动更新最终的页面。    

### 时间
每次的编译时间差不多在一分钟之内。    

![/images/2018_01_03_11_25_24_846x393.jpg](/images/2018_01_03_11_25_24_846x393.jpg)
