+++
title = "PlayingMySQLWorkbench"
date = "2017-11-15T16:45:28+08:00"
description = "playingmysqlworkbench"
keywords = ["Linux"]
categories = ["Linux"]
+++
### dump sql
Dump the existing sql via following commands:    

```
# mysqldump -uxxxx -pxxxxx table_name -h10.53.12.102 -P3306>xxxx.sql
```
### sql in docker
Run docker instance of sql via following command:    

```
$ sudo docker run --name=mysql1 -d -e MYSQL_ROOT_PASSWORD=xxxxxxx -v /media/sda5/mysql:/home -p3306:3306 mysql:5.7
$ sudo docker exec -it mysql1 /bin/sh
# mysql -uroot -pxxxx
> create database xxxxx
```
### import to sql
import sql files via following command:    

```
# mysql -uroot -pxxxxxx databasenamexxxxxx -h127.0.0.1 -P3306</media/sda5/mysql/xxxxxx.sql
```
### mysql-workbench
Install workbench via `sudo pacman -S mysql-workbench`, then start using
workbench for editing the database.    
