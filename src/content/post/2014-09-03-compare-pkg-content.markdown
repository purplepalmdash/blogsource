---
categories: ["Linux"]
comments: true
date: 2014-09-03T00:00:00Z
title: Compare pkg content
url: /2014/09/03/compare-pkg-content/
---

Use dpkg for reading the content and compare with the official ones:    

```
 dpkg -c ../../xxxxx_name.deb | awk '{print $3 $6}' | sort -n

```
Scripts for listing all of the content in the directory:    

```
for i in `ls *.deb`
do
    echo $i
    dpkg -c $i
done

```

