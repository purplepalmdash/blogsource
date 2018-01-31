+++
title = "OracleDatabaseDockerized"
date = "2018-01-31T14:24:19+08:00"
description = "OracleDatabaseDockerized"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Oracle In Docker
Reference as:    

[https://blog.ansheng.me/article/django-using-oracle-database.html?from=singlemessage&isappinstalled=0](https://blog.ansheng.me/article/django-using-oracle-database.html?from=singlemessage&isappinstalled=0)    

```
# firefox https://github.com/oracle/docker-images
### to get the docker-images-master.zip
# unzip docker-images-master.zip
# cd ./docker-images-master/OracleDatabase/dockerfiles/12.2.0.1
# firefox http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html
### to get linuxx64_12201_database.zip 
# ls linuxx64_12201_database.zip  -l -h
-rw-r--r-- 1 root root 3.3G Jan 29 22:31 linuxx64_12201_database.zip
# cd ..
# ls
11.2.0.2  12.1.0.2  12.2.0.1  buildDockerImage.sh
### edit following lines, ignore healthy check, or your building will be
### failed.
# vim 12.2.0.1/Dockerfile.ee
.....
VOLUME ["$ORACLE_BASE/oradata"]
EXPOSE 1521 5500
#HEALTHCHECK --interval=1m --start-period=5m \
#   CMD "$ORACLE_BASE/$CHECK_DB_FILE" >/dev/null || exit 1
.....
# ./buildDockerImage.sh -v 12.2.0.1 -e
```

After building , you will find the successfully-built image:    

```
# docker images | grep oracle
oracle/database                                                             12.2.0.1-se2        ace79efeb013        32 hours ago        13.3 GB
oracle/database                                                             12.2.0.1-ee         f9f11957b88d        39 hours ago        13.3 GB
```

### Upgraing from 11g
For using this docker image, do following:    

```
# docker run -d --name oracle -p 1521:1521 -p 5500:5500 -e ORACLE_SID=testsid
-e ORACLE_PWD=testpassword123 oracle/database:12.2.0.1-ee
# docker logs -f oracle
```
the instance will take around 1 minutes for using.    

System Configuration before importing sql: 

```
# docker exec -it oracle /bin/bash
[oracle@2e931f80a99a ~]$ ls
setPassword.sh
[oracle@2e931f80a99a ~]$ scp -r root@192.192.189.129:/root/fresh_install1 .
[oracle@2e931f80a99a ~]$ echo
"SQLNET.ALLOWED_LOGON_VERSION_SERVER=8">>/opt/oracle/product/12.2.0.1/dbhome_1/network/admin/sqlnet.ora
[oracle@2e931f80a99a ~]$ echo
"SQLNET.ALLOWED_LOGON_VERSION_CLIENT=8">>/opt/oracle/product/12.2.0.1/dbhome_1/network/admin/sqlnet.ora
[oracle@2e931f80a99a ~]$ exit
exit
# docker restart oracle
```

Importing sql:    

```
# docker exec -it oracle /bin/bash
[oracle@2e931f80a99a ~]$ export NLS_LANG="SIMPLIFIED CHINESE_CHINA.AL32UTF8"
[oracle@2e931f80a99a ~]$ sqlplus sys/testpassword123@localhost:1521/testsid as
sysdba

SQL*Plus: Release 12.2.0.1.0 Production on 星期三 1月 31 07:11:05 2018

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


连接到: 
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> @/home/oracle/fresh_install1/1_create_user_and_tablespace.sql
SP2-0734: 未知的命令开头 "﻿--查询表..." - 忽略了剩余的行。

会话已更改。


系统已更改。
```
Using newly created username/password for importing more sql files:    

```
[oracle@2e931f80a99a ~]$ sqlplus psm/psmpassword@localhost:1521/testsid

SQL*Plus: Release 12.2.0.1.0 Production on 星期三 1月 31 07:11:35 2018

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


连接到: 
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> @/home/oracle/fresh_install1/2_psm_ddl.sql
SQL> @/home/oracle/fresh_install1/3_psm_ddl.sql
SQL> @/home/oracle/fresh_install1/4_psm_ddl.sql
```

Testing using tord:    

![/images/2018_01_31_15_19_33_253x485.jpg](/images/2018_01_31_15_19_33_253x485.jpg)

Now you could using this created oracle db for testing purpose.    

### Explanation
Add following lines for using oracle 12c:   

Set following, or you won't create the user like 11g:    

```
alter session set "_ORACLE_SCRIPT"=true
```


Set following, or you won't logon to the system using toad:    

```
alter system set sec_case_sensitive_logon=false;
```
