+++
title= "koDatabase"
date = "2020-10-21T11:15:23+08:00"
description = "koDatabase"
keywords = ["Technology"]
categories = ["Technology"]
+++
Via following commands for recoving the user priviledge:    

```
# podman exec -it rong_mysql /bin/bash
```
Sql 操作:    

```
mysqlbash-4.2# mysql -uroot -p
Enter password: 
mysql> use ko
mysql> update ko_user set is_active=1 where name='admin';
mysql> update ko_user set is_admin=1 where name='admin;
```
Now  go back to login page, you will use admin user for login.
