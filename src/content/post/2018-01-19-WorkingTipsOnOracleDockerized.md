+++
title = "WorkingTipsOnOracleDBDockerized"
date = "2018-01-19T17:38:10+08:00"
description = "WorkingTipsOnOracleDBDockerized"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
Startup the container and enter it:    

```
# sudo docker create --name oraclexe11g01 -p 49160:22 -p 49161:1521 wnameless/oracle-xe-11g
# sudo docker start oraclexe11g01
# su - oracle
```

In Container, do following:    

```
oracle@c7edb0017fa8:~$ scp xxxxx@172.17.0.1:/media/sda5/fresh_install.tar.gz .
xxxxx@172.17.0.1's password: 
fresh_install.tar.gz                                                                                                        100%  135KB 135.2KB/s   00:00    
oracle@c7edb0017fa8:~$ tar xzvf fresh_install.tar.gz 
oracle@c7edb0017fa8:~$ cd fresh_install
oracle@c7edb0017fa8:~/fresh_install$ ls
1_create_user_and_tablespace.sql  2_xxxxxxxx_ddl.sql  3_xxxxxxxx_data.sql  4_xxxxxx.sql
```
Now create user and tablespaces:    

```
oracle@c7edb0017fa8:~/fresh_install$ export NLS_LANG="SIMPLIFIED CHINESE_CHINA.AL32UTF8"
oracle@c7edb0017fa8:~/fresh_install$ cd $ORACLE_HOME/bin
oracle@c7edb0017fa8:~/product/11.2.0/xe/bin$ ./sqlplus / as sysdba

SQL*Plus: Release 11.2.0.2.0 Production on 星期五 1月 19 07:53:32 2018

Copyright (c) 1982, 2011, Oracle.  All rights reserved.


连接到: 
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
SQL> @/u01/app/oracle/fresh_install/1_create_user_and_tablespace.sql
SP2-0734: 未知的命令开头 "﻿--查询表..." - 忽略了剩余的行。

表空间已创建。


用户已创建。
授权成功。

SQL> exit
从 Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production 断开
```
Now create your items via:    

```
oracle@c7edb0017fa8:~/product/11.2.0/xe/bin$ ./sqlplus xxxxxxxx/xxxxxxxx

SQL*Plus: Release 11.2.0.2.0 Production on 星期五 1月 19 08:02:13 2018

Copyright (c) 1982, 2011, Oracle.  All rights reserved.


连接到: 
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> @/u01/app/oracle/fresh_install/2_xxxxxxxx_ddl.sql

提交完成。
SQL> @/u01/app/oracle/fresh_install/3_xxxxxxxx_data.sql

SQL> @/u01/app/oracle/fresh_install/4_goegouwoguwog.sql


SQL> exit
从 Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production 断开
oracle@c7edb0017fa8:~/product/11.2.0/xe/bin$ exit
logout
root@c7edb0017fa8:~/fresh_install# exit
logout
Connection to localhost closed.
```

Save the docker image via:    

```
$ sudo docker commit c7edb0017fa8 mxxxxxxx
$ sudo docker save mxxxxxxx>mxxxxx.tar
```

OK, you got a initialized oracledb images.   
