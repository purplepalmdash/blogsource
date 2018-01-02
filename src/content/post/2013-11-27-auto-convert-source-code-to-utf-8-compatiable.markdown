---
categories: ["Linux"]
comments: true
date: 2013-11-27T00:00:00Z
title: Auto convert source code to UTF-8 compatiable
url: /2013/11/27/auto-convert-source-code-to-utf-8-compatiable/
---

For those code files which contains gb2312 or gbk format, we can use following scripts for automatically convert them to UTF-8 conpatiable format files. 

```
#!/bin/sh
export LANG="zh_CN.utf8"
export LC_ALL="zh_CN.utf8"
#for file in `find . -name "*.h"`
for file in `find . -name "*.c"`
do
	echo $file
	enca -L zh_CN -x UTF-8 $file
done

```
