---
categories: ["Pelican"]
comments: true
date: 2014-01-24T00:00:00Z
title: Update Qzone(2)
url: /2014/01/24/update-qzone-2/
---

Install pinyin for we want to use it for generate the title:

```
	$ pip install pinyin

```
Write a new post:

```
	$ python config_blog.py 迁移博客成功
	content/posts/2014/01/2014_01_24_qianyibokechenggong.md

```
Chinese codec:

```
	cat ~/.bashrc
	export LANG="zh_CN.UTF-8"              Or "en_US.UTF-8"
	export LC_ALL="zh_CN.UTF-8"          Or "en_US.UTF-8"

```
Write the blog:

```
	vim content/posts/2014/01/2014_01_24_qianyibokechenggong.md

```
Install BeautifulSoup

```
	pip install BeautifulSoup

```
	
	
