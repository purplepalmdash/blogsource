+++
title= "WorkingTipsOnDevOpsEnv"
date = "2020-07-06T08:51:24+08:00"
description = "WorkingTipsOnDevOpsEnv"
keywords = ["Technology"]
categories = ["Technology"]
+++
因为后续会基于这个框架来做开发，所以一开始就记录下搭建的有关事项，便于以后参考。    

### 基础环境
硬件/软件环境如下:   

```
虚拟机，4核10G内存，无swap分区
Ubuntu 20.04操作系统

```

安装docker:     

```
# systemctl stop apt-daily.timer;systemctl disable apt-daily.timer ; systemctl stop apt-daily-upgrade.timer ; systemctl disable apt-daily-upgrade.timer ; systemctl stop apt-daily.service ; systemctl mask apt-daily.service ; systemctl daemon-reload
# sudo apt-get install -y docker.io
# sudo docker version
19.03.8
# sudo apt-get install -y python3-dev sshpass default-libmysqlclient-dev krb5-config krb5-user  python3-pip libkrb5-dev
# sudo apt-get install -y  libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev unzip
```
创建数据库:    

```
# apt-get install -y mariradb-server
# MariaDB [(none)]> CREATE USER 'devops'@'localhost' IDENTIFIED BY 'devops';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON * . * TO 'devops'@'localhost';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> SHOW GRANTS FOR 'devops'@localhost;
+------------------------------------------------------------------------------------------------------------------------+
| Grants for devops@localhost                                                                                            |
+------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'devops'@'localhost' IDENTIFIED BY PASSWORD '*2683A2B8A9DE120C7B5CC6D45B5F7A2E708FAFCF' |
+------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.000 sec)

MariaDB [(none)]> create database devops;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> grant all privileges on devops.* TO 'devops'@'localhost' identified by 'devops';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> quit;

```

准备redis环境, guacd环境:    

```
# docker load<redis.tar
Loaded images: redis:alpine
# docker run --name redis-server -p 6379:6379 -d redis:alpine
# netstat -anp | grep 6379
tcp6       0      0 :::6379                 :::*                    LISTEN      9183/docker-proxy   
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
bf2d4dd7767f        redis:alpine        "docker-entrypoint.s…"   8 seconds ago       Up 5 seconds        0.0.0.0:6379->6379/tcp   redis-server
# docker pull guacamole/guacd:latest
# # docker run --name guacd -e GUACD_LOG_LEVEL=info -v  /home/vagrant/devops/media/guacd:/fs -p 4822:4822 -d guacamole/guacd
de109f5e821648bea88bb8cc08267934df09e87c113835118c956c05087f9c3b
root@leffss-1:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
de109f5e8216        guacamole/guacd     "/bin/sh -c '/usr/lo…"   4 seconds ago       Up 1 second         0.0.0.0:4822->4822/tcp   guacd
bf2d4dd7767f        redis:alpine        "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        0.0.0.0:6379->6379/tcp   redis-server
```
建立python3(3.8)虚拟环境:    

```
# apt-get install -y python3-venv
# python3 -m venv devops_venv
root@leffss-1:/home/vagrant/venv# source devops_venv/bin/activate
(devops_venv) root@leffss-1:/home/vagrant/venv# which python
/home/vagrant/venv/devops_venv/bin/python
# cat requirements.txt
django==2.2.11
supervisor==4.1.0
# simpleui
paramiko==2.6.0
decorator==4.4.2
gssapi==1.6.2
pyasn1==0.4.3
channels==2.2.0
channels-redis==2.4.0
celery==4.3.0
redis==3.3.11
eventlet==0.23.0
selectors2==2.0.1
django-redis==4.10.0
pyguacamole==0.8
# ansible==2.8.5
ansible==2.9.2
daphne==2.3.0
# gunicorn==19.9.0
gunicorn==20.0.3
gevent==1.4.0
async==0.6.2
pymysql==0.9.3
django-ratelimit==2.0.0
cryptography==2.7
mysqlclient==1.4.4
apscheduler==3.6.3
django-apscheduler==0.3.0
requests==2.22.0
jsonpickle==1.2
redisbeat==1.2.3
#  pip3 install -i https://mirrors.aliyun.com/pypi/simple -r requirements.txt
```
创建数据库:    

```
# sh delete_makemigrations.sh
# rm -f db.sqlite3
# # 以上是删除可能存在的开发环境遗留数据
# vim devops/settings.py
注释掉DATABASE相关的字段
# python3 manage.py makemigrations
# python3 manage.py migrate
```

![/images/2020_07_06_09_46_11_936x664.jpg](/images/2020_07_06_09_46_11_936x664.jpg)

初始化数据

```
python3 manage.py loaddata initial_data.json
python3 init.py
```
启动django服务:    

```
# mkdir -p logs
# export PYTHONOPTIMIZE=1		# 解决 celery 不允许创建子进程的问题
# nohup gunicorn -c gunicorn.cfg devops.wsgi:application > logs/gunicorn.log 2>&1 &
# nohup daphne -b 0.0.0.0 -p 8001 devops.asgi:application > logs/daphne.log 2>&1 &
# nohup python3 manage.py sshd > logs/sshd.log 2>&1 &
# nohup celery -A devops worker -l info -c 3 --max-tasks-per-child 40 --prefetch-multiplier 1 > logs/celery.log 2>&1 &
```
配置前端代理:    

```
# sudo apt-get install -y nginx
# Configure the nginx configuration.
```
Configuration files

```
# cat /etc/nginx/sites-enabled/default 
	upstream wsgi-backend {
		ip_hash;
		server 127.0.0.1:8000 max_fails=3 fail_timeout=0;
	}

	upstream asgi-backend {
		ip_hash;
		server 127.0.0.1:8001 max_fails=3 fail_timeout=0;
	}


server {
        listen 80 default_server;
		listen [::]:80 default_server;
		server_name  _;
		client_max_body_size 30m;
		add_header X-Frame-Options "DENY";
		
		location ~* ^/(media|static) {
			root /home/vagrant/devops-master;   # 此目录根据实际情况修改
			# expires 30d;
			if ($request_filename ~* .*\.(css|js|gif|jpg|jpeg|png|bmp|swf|svg)$)
			{
				expires 7d;
			}
		}

		location /ws {
			try_files $uri @proxy_to_ws;
		}

		location @proxy_to_ws {
			proxy_pass http://asgi-backend;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
			proxy_redirect off;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
			proxy_set_header X-Forwarded-Port $server_port;
			proxy_set_header X-Forwarded-Host $server_name;
			proxy_intercept_errors on;
			recursive_error_pages on;
		}

		location / {
			try_files $uri @proxy_to_app;
		}

		location @proxy_to_app {
			proxy_pass http://wsgi-backend;
			proxy_http_version 1.1;
			proxy_redirect off;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
			proxy_set_header X-Forwarded-Port $server_port;
			proxy_set_header X-Forwarded-Host $server_name;
			proxy_intercept_errors on;
			recursive_error_pages on;
		}

		location = /favicon.ico {
				 access_log off;    #关闭正常访问日志
		}

		error_page 404 /404.html;
		location = /404.html {
			root   /usr/share/nginx/html;
			if ( $request_uri ~ ^/favicon\.ico$ ) {    #关闭favicon.ico 404错误日志
				access_log off;
			}
		}

		error_page 500 502 503 504 /50x.html;
		location = /50x.html {
			root   /usr/share/nginx/html;
		}
	}
```
Now you could access the system.   
