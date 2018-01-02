---

comments: true
date: 2013-10-27T00:00:00Z
title: Automatically download all of the full-circle magazine
url: /2013/10/27/automatically-download-all-of-the-full-circle-magazine/
---

Following is a little script which could help you download all of the PDF version of the full-circle magazines:
```
#!/bin/bash
i=0
for i in {0..78}
do
	#echo $i
	i=`expr $i + 1`
	#http://dl.fullcirclemagazine.org/issue46_en.pdf
	url=`echo "http://dl.fullcirclemagazine.org/issue"$i"_en.pdf"`
	echo $url
	wget $url
done
```
If we change the url to CN version, then we can download all of the chinese version PDF:
```
#!/bin/bash
i=0
for i in {0..47}
do
	#echo $i
	i=`expr $i + 1`
	#http://dl.fullcirclemagazine.org/issue47_zh-CN.pdf
	url=`echo "http://dl.fullcirclemagazine.org/issue"$i"_zh-CN.pdf"`
	echo $url
	wget $url
done
```
