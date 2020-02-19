+++
title = "WorkingTipsOnPlayWithK8s"
date = "2018-04-12T09:13:49+08:00"
description = "WorkingTipsOnPlayWithK8s"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Aim
To Write an tutorial for colleagues for learning, they only have to open the
browser, by clicking then they could get an automated dev environment.   

### Environment
play-with-kubernetes blog:    

```
# git clone https://github.com/play-with-docker/play-with-kubernetes.github.io.git
# cd play-with-kubernetes.github.io/
# vim _config.yml
pwkurl: http://192.168.189.114
# docker-compose up
```
Then open your browser `http://192.168.189.114:4000`, and you will see the play-with-k8s webpages.   

For using the local infrastructure, to configure the `play-with-docker` with
following steps:    

```
# cd /root/go/src/github.com/play-with-docker/play-with-docker
# vim config/config.go
	//flag.StringVar(&DefaultDinDImage, "default-dind-image", "franela/dind", "Default DinD image to use if not specified otherwise")
	flag.StringVar(&DefaultDinDImage, "default-dind-image", "franela/k8s", "Default DinD image to use if not specified otherwise")
```
While the image we specified here could be the one you added your changes, but
default we will use `franela/k8s`.  

The webpage is showed as:    

![/images/2018_04_12_09_25_38_1242x708.jpg](/images/2018_04_12_09_25_38_1242x708.jpg)

     

 

