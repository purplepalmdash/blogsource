---

comments: true
date: 2013-09-20T00:00:00Z
title: 用python操纵Sqlite3
url: /2013/09/20/yong-pythoncao-zong-sqlite/
---

1\. 在命令行下使用sqlite3创建数据库, 运行完以下命令后，在本地目录会生成一个weather.db文件，由于我们什么都没有做，所以这个数据库现在是空的。
```
$ sqlite3 weather.db
SQLite version 3.8.0.2 2013-09-03 17:11:13
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .tables
sqlite> .exit
[Trusty@DashArch weather]$ ls
weather.db
```
2\. 

