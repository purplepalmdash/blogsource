+++
description = "Working tips on hugo template dimension"
title = "Working Tips On Dimension"
categories = ["Web"]
date = "2017-03-17T19:22:19+08:00"
keywords = ["Web"]

+++
### 背景
最近在帮一个亲戚做企业类网站，最初方案是基于Docker化的WordPress，搭建好以后
一直不闻不问，思想可能是因为WP对于小白用户来说太过于复杂的缘故。

正好今天查看hugo的模板方案时，发现有一个很美观的静态页面叫Dimension, 于是起
了把网站迁移到静态网站上的心思。    

Dimension主题预览:    

![/images/2017_03_17_19_27_38_439x324.jpg](/images/2017_03_17_19_27_38_439x324.jpg)

### 搭建环境
首先从github下载源代码:    

```
$ git clone https://github.com/sethmacleod/dimension.git
$ mkdir Website/themes
$ mv dimension-master Website/themes/dimension
$ cp -ar Website/themes/dimension/exampleSite/* Website/
```
经过上面的步骤，我们已经设置了模板可工作的DEMO环境，在Website目录下执行以下命令，
分别为编译整个静态网站和预览更改:    

```
$ cd Website
$ hugo
$ sudo python2 -mSimpleHTTPServer 18118
```
现在打开你的浏览器访问`http://localhost:18118`，即可看到本地搭建的dimension示例网站。    

### 自定义
如果要创建自己的页面，则`hugo new
Your-Page.md`即可创建出来页面，编辑方法就是针对markdown的编辑。    

#### 多语言支持
默认的DEMO提供了对德语和英语的支持，我们只需要删除`config.toml`文件中的`[languages]`的字段就可以,
例如删除:    

```
[languages]
[languages.en]
languageName = "English"
weight = 1
title = "Dimension"
```
#### logo修改
默认的logo使用的是fontawesome字体，我们可以修改为自定义的图片，需要修改以下两个地方:    
config.toml文件:    

```
- logo = "fa-diamond"
- logo = "/images/jqlogo.png"
```
同时放置`jqlogo.png`文件于`static/images/`目录下.    

`themes/dimension/layouts/index.html`文件中需要做对应的修改:    

```
            {{ with .Site.Params.logo }}            
-              <span class="icon {{ . }}"></span>
+	    <img src="{{ . }}" style="padding: 10px 10px 10px 10px;" width="100%">
```
这样就完成了对logo的修改.    

#### 首页滚动字幕
这里主要参考了另一个网站`https://robinson-schule.ch/`中的解决方案。    

```
$ cd themes/dimension/static/js
$ wget https://robinson-schule.ch/js/quotes.js
$ cd ../css
$ wget https://robinson-schule.ch/css/project.css
```
接着就可以修改`quotes.js`文件中关于滚动条的内容，同时在`themes/dimension/layouts/index.html`文件
中添加:    

```
                <p>{{ .Site.Params.description | safeHTML }}</p>
+		<p id="quotes"></p><script type="text/javascript" src="/js/quotes.js"></script></p>
```

#### 人物介绍
markdown中可以直接添加html，添加以下的html字段来实现图像的自动圆圈处理:    

```
  <div class="person">
	<div class="personheadshot">
		<img src="/images/yuangong3.jpg" alt="yuangong3" class="circle headshot">
	</div>
	<div class="persondescription">
		<span><b>刘梦之</b></span><br/>
		<span><i>场地保洁主管 / 场地保洁专家 / 某保洁项目组组长</i></span><br/>
		<span>晶巧连续四年优秀员工 (2013 / 2014 / 2015 / 2016)</span>
	</div>
  </div>
```
这里用到的`circle headshot`的css定义在`project.css`文件中:    

```
img.headshot
{
    width: 150px;
    height: 150px;
}
img.circle {
 	border-radius: 50%;
}
```
而`div.person`等定义也可以在该文件中找到。    

### 完成效果
首页:    

![/images/2017_03_17_20_06_26_757x605.jpg](/images/2017_03_17_20_06_26_757x605.jpg)

about页面:    

![/images/2017_03_17_20_06_55_649x482.jpg](/images/2017_03_17_20_06_55_649x482.jpg)

Enjoy it!!!

### 延伸阅读
可以考虑hugo的另一个模板效果:    

[http://themes.gohugo.io/theme/hugo-elate-theme/](http://themes.gohugo.io/theme/hugo-elate-theme/)    
