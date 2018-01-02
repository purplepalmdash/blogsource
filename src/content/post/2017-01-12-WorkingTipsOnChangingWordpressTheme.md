+++
title = "Wordpress长图片功能实现"
date = "2017-01-12T12:06:15+08:00"
categories = ["Linux"]
keywords = ["Linux"]
description = "For changing wordpress template"

+++
### 目标
在Wordpress的每篇文章添加TurnToJPG功能，点击该按钮以后，由博文直接生成长文字图片。

### 搭建测试环境
为了及时测试我们的博客，快速搭建一个基于docker的开发环境:    

```
$ sudo docker pull wordpress
$ sudo docker pull mariadb
$ sudo docker pull corbinu/docker-phpmyadmin
```
创建一个docker-compose的yml文件，启动之:    

```
$ vim docker-compose.yml 
wordpress:
  image: wordpress
  links:
    - wordpress_db:mysql
  ports:
    - 8080:80
wordpress_db:
  image: mariadb
  environment:
    MYSQL_ROOT_PASSWORD: examplepass
phpmyadmin:
  image: corbinu/docker-phpmyadmin
  links:
    - wordpress_db:mysql
  ports:
    - 8181:80
  environment:
    MYSQL_USERNAME: root
    MYSQL_ROOT_PASSWORD: examplepass
```
现在启动服务:    

```
$ sudo docker-compose up -d
Creating mywordpress_wordpress_db_1
Creating mywordpress_wordpress_1
Creating mywordpress_phpmyadmin_1
```
打开`http://127.0.0.1:8080`即可访问到我们的测试站点。在wordpress后台
查找并安装`tdPersona`主题.    

### 分析
#### 所用主题模板
随机拷贝某篇博文的html, 而后:    

```
$ cat test.html | grep -i themes
<link rel='stylesheet' id='tdpersona-icons-css'  href='http://www.hahahahohoho.com.cn/wp-content/themes/tdpersona/css/font-awesome.min.css?ver=4.6.2' type='text/css' media='all' />
<link rel='stylesheet' id='tdpersona-framework-css'  href='http://www.hahahahohoho.com.cn/wp-content/themes/tdpersona/css/bootstrap.min.css?ver=4.6.2' type='text/css' media='all' />
<link rel='stylesheet' id='style-css'  href='http://www.hahahahohoho.com.cn/wp-content/themes/tdpersona/style.css?ver=4.6.2' type='text/css' media='all' />
```
可见原博主选用的是`tdpersona`主题.    

#### 找到需要图形化的部分
在chromium里按`F12`, 耐心找寻你需要的那一部分。   

![/images/2017_01_12_19_54_51_636x338.jpg](/images/2017_01_12_19_54_51_636x338.jpg)    

我的:    

```
<article id="post-10746" class="post-10746 post type-post status-publish format-standard hentry category-1">
// .........
</article>
```
进一步看，发现我们需要在`<head></head>`动文章:    

![/images/2017_01_12_12_21_05_626x149.jpg](/images/2017_01_12_12_21_05_626x149.jpg)    

```
	<header class="entry-header">
		<h2 class="entry-title"><?php the_title(); ?></h2>

		<?php if ( 'post' == get_post_type() ) : ?>
			<?php tdpersona_post_date(); ?>
		<?php endif; ?>

		<?php if ( has_post_thumbnail() ): ?>
		<div class="post-thumb">
			<?php the_post_thumbnail(); ?>
		</div><!-- .post-thumb -->
		<?php endif; ?>

	</header><!-- .entry-header -->
```
### 代码修改
`/var/www/html/wp-content/themes/tdpersona/template-parts/content-single.php`:    

首尾分别添加如下行数(+代表添加的行数):   

```
+ <div class="capturepost">
<article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
	</footer><!-- .entry-meta -->
</article><!-- #post -->
+ </div>
```

`/var/www/html/wp-content/themes/tdpersona/inc/template-tags.php`:    

```
	$output .= '<div class="entry-date">';
	$output .= '<a href="'.esc_url( get_permalink() ).'">'.$date_format_html.'</a>';
	$output .= '</div><!-- .entry-date -->';

+	$output .= '<div class="save-to-jpg", align="right">';
+	$output .= '<a href="javascript:genPostShot()">TurnToJPG --> <i class="fa fa-camera-retro fa-2x"></i></a><a id="test"></a>';
+	$output .= '</div><!-- .save-to-jpg -->'; 
+	$output .= '<hr>';
```
这将在标题栏下面添加一个图标和链接，点击该图标将触发javascript函数，将当前页面
转变为jpg图片.    

`/var/www/html/wp-includes/formatting.php`:    

```
		<script type="text/javascript">
			window._wpemojiSettings = <?php echo wp_json_encode( $settings ); ?>;
			!function(a,b,c){function d(a){var b,c,d,e,f=String.fromCharCode;if(!k||!k.fillText)return!1;switch(k.clearRect(0,0,j.width,j.height),k.textBaseline="top",k.font="600 32px Arial",a){case"flag":return k.fillText(f(55356,56826,55356,56819),0,0),!(j.toDataURL().length<3e3)&&(k.clearRect(0,0,j.width,j.height),k.fillText(f(55356,57331,65039,8205,55356,57096),0,0),b=j.toDataURL(),k.clearRect(0,0,j.width,j.height),k.fillText(f(55356,57331,55356,57096),0,0),c=j.toDataURL(),b!==c);case"emoji4":return k.fillText(f(55357,56425,55356,57341,8205,55357,56507),0,0),d=j.toDataURL(),k.clearRect(0,0,j.width,j.height),k.fillText(f(55357,56425,55356,57341,55357,56507),0,0),e=j.toDataURL(),d!==e}return!1}function e(a){var c=b.createElement("script");c.src=a,c.defer=c.type="text/javascript",b.getElementsByTagName("head")[0].appendChild(c)}var f,g,h,i,j=b.createElement("canvas"),k=j.getContext&&j.getContext("2d");for(i=Array("flag","emoji4"),c.supports={everything:!0,everythingExceptFlag:!0},h=0;h<i.length;h++)c.supports[i[h]]=d(i[h]),c.supports.everything=c.supports.everything&&c.supports[i[h]],"flag"!==i[h]&&(c.supports.everythingExceptFlag=c.supports.everythingExceptFlag&&c.supports[i[h]]);c.supports.everythingExceptFlag=c.supports.everythingExceptFlag&&!c.supports.flag,c.DOMReady=!1,c.readyCallback=function(){c.DOMReady=!0},c.supports.everything||(g=function(){c.readyCallback()},b.addEventListener?(b.addEventListener("DOMContentLoaded",g,!1),a.addEventListener("load",g,!1)):(a.attachEvent("onload",g),b.attachEvent("onreadystatechange",function(){"complete"===b.readyState&&c.readyCallback()})),f=c.source||{},f.concatemoji?e(f.concatemoji):f.wpemoji&&f.twemoji&&(e(f.twemoji),e(f.wpemoji)))}(window,document,window._wpemojiSettings);
		</script>
+		/* Actually add it into here seems not a good idea, just a hacking */
+		<script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
+		<script type="text/javascript" src="/js/html2canvas.js"></script>
+		<script type='text/javascript'>//<![CDATA[
+		function genPostShot() { 
+		        var rightNow = new Date();
+		        var imageName = rightNow.toISOString().slice(0,16).replace(/(-)|(:)|(T)/g,"");
+		        imageName += '.jpg'
+		        html2canvas(document.getElementsByClassName('capturepost'), {
+		            background :'#FFFFFF',
+		            onrendered: function(canvas) {
+				// Click them for download
+		        	$('#test').attr('href', canvas.toDataURL("image/jpeg"));
+		        	$('#test').attr('download',imageName);
+		        	$('#test')[0].click();
+		            }
+		        });
+		}; 
+		//]]>
+		</script>
+		/* Hacking Ended */

```
其实这个hacking做得不好，整洁的代码应该是新建一个函数用于添加这些代码。    

最后，在`/var/www/html`下创建一个文件夹，并下载html2canvas.js文件:    

```
$ sudo mkdir -p /var/www/html/js
$ cd js
$ sudo wget https://cdn.bootcss.com/html2canvas/0.5.0-alpha1/html2canvas.js
```
或者直接修改上面定义中的`/js/html2canvas.js` 为:    

```
//cdn.bootcss.com/html2canvas/0.5.0-alpha1/html2canvas.js
```

### 验证
现在重新刷新博客页面，即可享受一键另存为图片的快捷功能。

![/images/2017_01_12_19_19_28_665x330.jpg](/images/2017_01_12_19_19_28_665x330.jpg)    
