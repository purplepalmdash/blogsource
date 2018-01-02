+++
categories = ["Linux"]
date = "2016-09-08T12:00:39+08:00"
description = "On youtube download"
keywords = ["Linux"]
title = "OnYoutube dl"

+++
### Background
If you download a playlist on youtube, your download files would be renamed as
following:    

```
$ ls -l -h /mnt/golang
total 4.8G
-rwxr--r-- 1 dash root  62M Sep  7 18:43 'Aerospike Install Linux-bw0eipI7-4s.mp4'
-rwxr--r-- 1 dash root 9.8M Sep  7 22:48 'App Enginge Domains-rNI_PyNuS2o.mp4'
-rwxr--r-- 1 dash root  28M Sep  7 21:52 'General Overview of Networking & The
Internet-hZ7cX4fpMk4.mp4'
-rwxr--r-- 1 dash root  51M Sep  7 22:25 'Golang Aerospike-symvVMJlC3g.mp4'
-rwxr--r-- 1 dash root 134M Sep  7 20:19 'Golang AJAX-UkEuYXi36o8.mp4'
```
### Solution
Manually download the playlist.txt from the webpage, something like:    

```
$ cat /mnt/golang/playlist.txt
Who uses golang
Installing Golang
Golang Webstorm & Golang Atom
Golang Hello World - Part 1 of 2
Golang Hello World - Part 2 of 2
Golang variables and scope
Golang Go Get
```
Use following shell script for rename all of the downloaded files:    

```
$ cat rr.sh
#!/bin/bash
i=0
while IFS='' read -r line || [[ -n "$line" ]]; do
    let i++
    prefix=`printf "%04d-" $i`
    content=`ls | grep "$line-"`
    cp "$content" ../golang1/"$prefix$line".mp4
done < ./playlist.txt
```

Your renamed files will be in the ../golang1 directory, now the name is listed as:    

```
$ ls -l ../golang1   
total 4984936
-rwxr--r-- 1 dash root  28352757 Sep  8 11:52 '0001-Who uses golang.mp4'
-rwxr--r-- 1 dash root  51195540 Sep  8 11:52 '0002-Installing Golang.mp4'
-rwxr--r-- 1 dash root  27507302 Sep  8 11:52 '0003-Golang Webstorm & Golang Atom.mp4'
-rwxr--r-- 1 dash root  22430385 Sep  8 11:52 '0004-Golang Hello World - Part 1 of 2.mp4'
-rwxr--r-- 1 dash root  37552977 Sep  8 11:52 '0005-Golang Hello World - Part 2 of 2.mp4'
-rwxr--r-- 1 dash root  27800500 Sep  8 11:52 '0006-Golang variables and scope.mp4'
-rwxr--r-- 1 dash root  19457050 Sep  8 11:52 '0007-Golang Go Get.mp4'
```
