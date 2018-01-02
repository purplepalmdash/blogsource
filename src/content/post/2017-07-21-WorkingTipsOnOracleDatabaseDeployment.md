+++
title = "WorkingTipsOnOracleDatabaseDeployment"
description = "WorkingTipsOnOracleDatabaseDeployment"
date = "2017-07-21T12:28:35+08:00"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Items
Working items on one-click deployment of oracle database.     

### Ansible-Playbooks
Based on:   

[https://github.com/nkadbi/oracle-db-12c-vagrant-ansible](https://github.com/nkadbi/oracle-db-12c-vagrant-ansible)    

Refers to:    

[https://blog.dbi-services.com/vagrant-up-get-your-oracle-infrastructure-up-and-running/](https://blog.dbi-services.com/vagrant-up-get-your-oracle-infrastructure-up-and-running/)    
[https://blog.dbi-services.com/part2-vagrant-up-get-your-oracle-infrastructure-up-an-running/](https://blog.dbi-services.com/part2-vagrant-up-get-your-oracle-infrastructure-up-an-running/)    

Username/Password:    
System: oracle/welcome1    
Database: sys/oracle    

### Linux Client
Yaourt has the linux client for accessing oracle Db:   

[https://aur.archlinux.org/packages/oracle-sqldeveloper/](https://aur.archlinux.org/packages/oracle-sqldeveloper/)    

Installing method:   
Download the file from oracle.com

### Create Database
Create database using following command:    

```
[vagrant@dbserver1 ~]$ su - oracle
Password: 
-bash-4.2$ sqlplus "/as sysdba"
```
Now you got the shell like `SQL>`, you could input the sql in this shell:     

```
Run `1_create_user_and_tablespace_dash.sql`
```
### Create tables/metadatas
The first step will create the database user, then you could login into the
database using this user, using SQL Devloper for login and execute the
command:    

![/images/2017_07_23_13_47_44_745x382.jpg](/images/2017_07_23_13_47_44_745x382.jpg)

Execute the following script:    

```
msp_XXX.sql(Including 2 scripts)   
```

![/images/2017_07_23_13_50_23_506x466.jpg](/images/2017_07_23_13_50_23_506x466.jpg)

Tips for getting the db config:    

```
 SQL> show parameter service_names;
.....
service_names			     string	 db1.private
```
Then your configuration should use the same `service_names` as described.    
