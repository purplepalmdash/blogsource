+++
categories = ["Linux"]
date = "2016-07-18T14:53:58+08:00"
description = "Use Jenkins for building packer.io"
keywords = []
title = "使用Jenkins/PackerIO自动化编译虚拟机镜像"

+++
### GitLab仓库
在前面搭建的GitLab里创建一个新仓库，用于存储Packer.io脚本。    

![/images/2016_07_18_14_57_26_995x911.jpg](/images/2016_07_18_14_57_26_995x911.jpg)     
在编译机器的仓库里，运行以下命令，添加自己到新创建的仓库里:     

```
$ cd existing_folder
$ git init
$ git remote add origin http://192.168.1.79:10080/root/BuildUbuntu.git
$ git add .
$ git commit
$ git push -u origin master
```
提交完毕之后，在GitLab服务器上就可以看到新添加的代码了。      

### Jenkins配置
在Jenkins里创建一个新项目，选择`Freestyle Project`， 默认创建完毕。    

在源代码管理的设置中，填入以下的条目:     

![/images/2016_07_18_15_07_28_1366x467.jpg](/images/2016_07_18_15_07_28_1366x467.jpg)   

`Build Trigger`中我们选择由GitLab触发：    

![/images/2016_07_18_15_13_31_1154x436.jpg](/images/2016_07_18_15_13_31_1154x436.jpg)    

在GitLab中我们需要添加相应的钩子(WebHook):     

![/images/2016_07_18_15_15_11_452x602.jpg](/images/2016_07_18_15_15_11_452x602.jpg)     
设置:     

![/images/2016_07_18_15_16_30_1044x427.jpg](/images/2016_07_18_15_16_30_1044x427.jpg)    

添加Build脚本(选择execute shell):    

![/images/2016_07_18_15_24_00_358x581.jpg](/images/2016_07_18_15_24_00_358x581.jpg)       

配置完毕后，就可以通过点击`Build Now`来编译Packer.io工程了。 
