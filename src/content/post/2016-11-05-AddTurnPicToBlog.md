+++
categories = ["Linux"]
date = "2016-11-05T18:28:54+08:00"
description = "Add turn to picture for blogging"
keywords = ["Linux"]
title = "为Blog添加保存为图片功能"

+++
### 背景
众所周知疼讯其实一直是一个很封闭傻逼的企业，早年QZONE的代码就层层加密，
惟恐互联网爬虫爬到它上面取内容。进入到移动互联网时代，就更加变本加厉，盆友
圈里的诸多内容，恐怕诸位都是没办法弄出来的。    

早年曾在Blog上添加过`分享到朋友圈`或`分享到微信好友`按钮，由于`github.io`未
被疼讯认可，共享出来的链接通常只有自己能看到，这又是一个拿用户当SB的典型
案例。

还好，对于用户上传图片，疼讯是没法限制的。所以有了以下的开发。     

### 功能参考
在博文里添加按钮`一键保存为图片`,即可将该篇文章生成为长图片。   

![/images/2016_11_05_18_38_31_637x390.jpg](/images/2016_11_05_18_38_31_637x390.jpg)

参考设计： 锤子便签。顺便吐槽一下锤子便签，添加了关键词过滤，关键词嘛，这也是
党国拿来折腾屁民的一个傻逼玩意，例如有一首歌是这么唱的:    

```
我爱北京关键词，
关键词上太阳升。。
伟大得领袖关键词，
带领我们向前进…
```

 ╮〔╯ε╰〕╭   ---太多的关键词，容易让写博文的人心情不好。

### 可选方案
A. wkhtmltoimage.    
这个转换起来是全屏转换，所以最初想到的是对网页做再加工，抠取出其中含有内容的一部分。
好处是，真的是所见即所得。     

```
$ wkhtmltoimage http://www.google.com google.jpg
```

B. html2canvas.    
这种方案直接在html页面上做文章，添加按钮后直接用画布来生成对象。缺点是有些格式显示出来
有少量阴影。    

暂时基于方案B实现。    

### Code
参考资料:    
[https://www.youtube.com/watch?v=-d2FeFiBvEo](https://www.youtube.com/watch?v=-d2FeFiBvEo)    
[https://html2canvas.hertzen.com/](https://html2canvas.hertzen.com/)    

因为本博客基于hugo构建，直接修改hugo使用的主题即可添加html2canvas。    

目录结构:    

```
$ ls  
config.toml  content  id_rsa.enc  scripts  static  themes
$ themes/hyde-a
$ ls
archetypes  images  layouts  LICENSE  README.md  static  theme.toml
```
下载`html2canvas.js`到主题的`static/js`目录:   

```
$ cd static/js
$ wget
https://github.com/niklasvh/html2canvas/releases/download/0.4.1/html2canvas.js
```
在主题的`./layouts/partials/head.html`文件中添加所需要的js文件及javascript代码
, (+代表新添加代码):     


```javascript
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.5/styles/docco.min.css">
+  <script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
+  <script type="text/javascript" src="/js/html2canvas.js"></script>
+  <script type='text/javascript'>//<![CDATA[
+  function genPostShot() { 
+          var rightNow = new Date();
+          var imageName = rightNow.toISOString().slice(0,16).replace(/(-)|(:)|(T)/g,"");
+          imageName += '.jpg'
+          html2canvas(document.getElementsByClassName('post'), {
+              onrendered: function(canvas) {
+  		// Click them for download
+          	$('#test').attr('href', canvas.toDataURL("image/jpeg"));
+          	$('#test').attr('download',imageName);
+          	$('#test')[0].click();
+              }
+          });
+  }; 
+  //]]>
+  </script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.5/highlight.min.js"></script>
```
简单解释一下这几行代码：    
1. 取当时UTC时间作为下载jpg文件名。    
2. 在当前页面中根据div的class名称找到`post`字段，构建一个画布。      
3. 将画布内容输出为jpeg文件，触发浏览器的一个下载动作。    

**注意**: `#test`为在html文件中添加的一个不可见链接.    

接下来修改每个博文用到的模板文件(layourts/post/single.html):    

```
{{ partial "head.html" . }}
<div class="content container">
  <div class="post">
    <h1>{{ .Title }}</h1>
+    <p align="right">
+    <a href="javascript:genPostShot()">
+	    TurnToJPG --> <i class="fa fa-camera-retro fa-2x"></i>
+    </a>
+    <a id="test"></a>
+    </p>
+    <hr>
    <span class="post-date">{{ .Date.Format "Jan 2, 2006" }} {{ if .Site.DisqusShortname }} &middot; <a href="{{ .Permalink }}#disqus_thread">Comments</a>{{ end }}
```
简单解释一下上述添加的代码:    

添加一个超链接，指向上文定义的javascript函数`genPostShot()`.    

保存修改，提交到上游仓库，重新生成整个静态网站，现在每个博文里都含有保存图片按钮。    
### 效果
譬如，点击[http://purplepalmdash.github.io/2013/11/17/run-in-winter/](http://purplepalmdash.github.io/2013/11/17/run-in-winter/)    

效果如下:   

![/images/2016_11_05_18_56_04_638x402.jpg](/images/2016_11_05_18_56_04_638x402.jpg)    

点击按钮，下载为文件201611051057.jpg(当前UTC时间戳），效果如下:   

![/images/2016_11_05_18_58_11_250x375.jpg](/images/2016_11_05_18_58_11_250x375.jpg)    

打完，收工，以后在盆友圈想怎么咆哮就可以怎么咆哮了。

### Trouble-Shooting
之所以加这么个条目是因为，在未定义背景的网页里，有时候截出来的图是黑色背景的，修正：    

修改`./static/js/html2canvas.js`文件:    

```
sTransparent = function(backgroundColor) {
  return (backgroundColor === "transparent" || backgroundColor === "rgba(0, 0,
0, 0)");
};
```
改为:    

```
_html2canvas.Util.isTransparent = function(backgroundColor) {
  return (backgroundColor === "transparent" || backgroundColor === "rgba(0, 0,
0, 0)" || backgroundColor === undefined);
};
```
之后我们在`./layouts/partials/head.html`中，添加这一行:    

```
          html2canvas(document.getElementsByClassName('post'), {
+              background :'#FFFFFF',
              onrendered: function(canvas) {
```
现在生成的图片背景就是白色的了。    
