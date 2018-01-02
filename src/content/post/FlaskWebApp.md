+++
categories = ["Web"]
date = "2016-07-12T15:26:23+08:00"
description = "Web Application Flask"
keywords = []
title = "用Flask框架搭建Web App"

+++
拥有Web界面的好处是显而易见的，譬如说，我们可能需要下载Youtube上的某一段视频。传统的操
作方式是这样的：登录位于国外的vps-> 下载youtube视频到VPS -> 退出vps登录 ->采用某种手段
(ftp/scp?)传送到本地。这时候如果有一个运行于远端VPS上的Web App，本地输入Youtube视频链接
，下载完毕后直接生成下载链接，这该有多好！这里我们来实现这个功能。      

### Flask
#### 运行环境
远程VPS位于Digital Ocean上，运行Ubuntu14.04。 这里我们基于python virtualenv来构建Flask
开发框架.     

```
$ sudo apt-get install -y python-virtualenv
$ virtualenv myflask
$ source ~/myflask/bin/activate
$ mkdir ~/flask_youtube
$ vim requirements.txt
Flask==0.10.1
$ pip install -r requirements.txt
```
运行完上述命令后，flask运行环境就已经就绪了。     
#### Flask App
这里我们参考了以下链接（实际上是照搬):    
[http://charlesleifer.com/blog/a-flask-front-end-and-chrome-extension-for-youtube-dl/](http://charlesleifer.com/blog/a-flask-front-end-and-chrome-extension-for-youtube-dl/)     

也参考了(关于virtualenv):     
[https://realpython.com/blog/python/setting-up-a-simple-ocr-server/](https://realpython.com/blog/python/setting-up-a-simple-ocr-server/)     

```
$ vim youtube.py
import subprocess
import sys

from flask import Flask, flash, redirect, request, render_template, url_for

DEBUG = False
SECRET_KEY = 'this is needed for flash messages'

BINARY = '/usr/bin/youtube-dl'
DEST_DIR = '/home/dash/videos'
OUTPUT_TEMPLATE = '%s/%%(title)s-%%(id)s.%%(ext)s' % DEST_DIR

app = Flask(__name__)
app.config.from_object(__name__)

@app.route('/', methods=['GET', 'POST'])
def download():
    if request.method == 'POST':
        url = request.form['url']
        p = subprocess.Popen([BINARY, '-o', OUTPUT_TEMPLATE, '-q', url])
        p.communicate()
        flash('Successfully downloaded!', 'success')
        return redirect(url_for('download'))
    return render_template('download.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8801)
```
实例中用到了flask里的模版，为此我们需要创建template目录:     

```
$ mkdir templates
$ vim templates/download.html
<!doctype html>
<html>
  <head>
    <title>youtuber</title>
    <link rel=stylesheet type=text/css href="{{ url_for('static', filename='css/youtuber.min.css') }}" />
    <script src="{{ url_for('static', filename='js/jquery-1.11.0.min.js') }}" type="text/javascript"></script>
    <script src="{{ url_for('static', filename='js/bootstrap.min.js') }}"></script>
  </head>
  <body>
    <div class="container">
      {% for category, message in get_flashed_messages(with_categories=true) %}
        <div class="alert alert-{{ category }} alert-dismissable">
          <button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
          <p>{{ message }}</p>
        </div>
      {% endfor %}

      <h1>Download</h1>
      <form action="{{ url_for('download') }}" method="post">
        <div class="form-group">
          <label for="url">URL</label>
          <input type="text" name="url" class="form-control" id="url" placeholder="url" />
        </div>
        <button type="submit" class="btn btn-primary">Download</button>
      </form>
    </div>
  </body>
</html> 
```
因为我们这里有指定用`static/js`下的静态文件，所以需要手动创建目录并下载文件:    

```
$ mkdir -p static/js
$ cd static/js
$ wget https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css
$ wget https://code.jquery.com/jquery-1.11.0.js
```
`youtuber.min.css`是样式表文件，这里先不加入，除了效果差一点，没啥副作用。    
#### 运行
键入`python youtube.py`即可运行该Web App. 访问`http://YourIPAddress:8801`即可访问到该WebApp的界面:     

![/images/2016_07_12_15_59_39_272x160.jpg](/images/2016_07_12_15_59_39_272x160.jpg)    

输入下载链接后，点击submit按钮后，VPS将自动下载该youtube影片。    


### 下载结果页面
下载结果页面需要在static目录下新建一个用于存放视频的`videos/`目录夹. 同时需要添加一个模
板文件.     

```
$ tree
.
└── down
    ├── requirements.txt
    ├── static
    │   ├── js
    │   │   ├── bootstrap-3.0.0.min.js
    │   │   ├── bootstrap.min.js
    │   │   ├── jquery-1.10.2.min.js
    │   │   └── jquery-1.11.0.min.js
    │   └── videos
    │       ├── abc.mp4
    │       └── ccc.mp4
    ├── templates
    │   ├── download.html
    │   └── videos.html
    └── youtube.py
```
其中`templates/videos.html`文件是用于渲染下载视频文件的模版，内容如下:    

```
<!doctype html>
<html>
  <head>
    <title>Video download info</title>
  </head>
  <body>
    <div class="container">
      <h1>All Downloadable Videos</h1>
      {% for video in video_url %}
      <a href="/static/videos/{{ video }}">{{ video }}</a><br />
      {% endfor %}
      <h1>Click Following Links For Downloading!</h1>
      <a href="/">Go Back To Download Page</a><br />
    </div>
  </body>
</html>
```

在`youtube.py`文件中添加`/videos`路由，及相关处理函数:    

```
import subprocess
import sys
import os
from flask import Flask, flash, redirect, request, render_template, url_for

DEBUG = False
SECRET_KEY = 'this is needed for flash messages'

BINARY = '/usr/bin/youtube-dl'
DEST_DIR = '/home/dash/down/static/videos/'
OUTPUT_TEMPLATE = '%s/%%(title)s-%%(id)s.%%(ext)s' % DEST_DIR

app = Flask(__name__)
app.config.from_object(__name__)

@app.route('/', methods=['GET', 'POST'])
def download():
    if request.method == 'POST':
        url = request.form['url']
        print url
        p = subprocess.Popen([BINARY, '-o', OUTPUT_TEMPLATE, '-q', url])
        p.communicate()
        flash('Successfully downloaded!', 'success')
        return redirect(url_for('videos'))
    return render_template('download.html')

# For holding all of the videos
@app.route('/videos')
def videos():
    names = os.listdir(os.path.abspath(DEST_DIR))
    return render_template('videos.html', video_url=names)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8801)
```

### 下载流程
首先，访问`http://YourIpAddress:8801/`, 得到的结果如下:    

![/images/2016_07_12_19_34_59_309x177.jpg](/images/2016_07_12_19_34_59_309x177.jpg)    

填入youtube上的视频地址以后，下载完毕后，可以看到videos页面变成了:    

![/images/2016_07_12_19_38_03_582x271.jpg](/images/2016_07_12_19_38_03_582x271.jpg)    

### 删除功能
`youtube.py`添加一个函数和路由:     

```
# For deleting specified file
@app.route('/delete/<filename>')
def remove_file(filename):
    filename_full = os.path.join(DEST_DIR, filename)
    print filename_full
    os.remove(filename_full)
    return redirect(url_for('videos'))
```
模版文件更改:    

```
$ vim templates/videos.html 
<!doctype html>
<html>
  <head>
    <title>Video download info</title>
  </head>
  <body>
    <div class="container">
      <h1>All Downloadable Videos</h1>
      {% for video in video_url %}
      <a href="/static/videos/{{ video }}">{{ video }}</a>
      <a href="/delete/{{ video }}"><img src="/static/img/fuck.png" alt="FuckYou"></a>
      <a href="/delete/{{ video }}"> Delete this video</a>
      <br />
      <hr>
      {% endfor %}
...........
```

`fuck.png`是从网上下载的图片，更改完毕后，页面如下:      
![/images/2016_07_12_20_27_39_748x382.jpg](/images/2016_07_12_20_27_39_748x382.jpg)    

### ToDo
添加权限，只有认证过后的用户才能使用删除功能。    
