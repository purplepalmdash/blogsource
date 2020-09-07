+++
title= "Workingtiponaarchftp"
date = "2020-09-07T09:28:51+08:00"
description = "Workingtiponaarchftp"
keywords = ["Technology"]
categories = ["Technology"]
+++
### ftpd服务端
加载镜像:    

```
# docker load<ftpd.tar
Loaded image: gists/pure-ftpd:arm64
# docker images | grep ftpd
gists/pure-ftpd                                                            arm64               1b3e76d8756b        3 months ago        5.77MB
```

运行以下命令, 创建一个vsftpd实例, 当前目录下的ftpd含有ftpd的配置文件(pureftpd)及存储目录(data):    

```
# mkdir ftpd
# cd ftpd
# mkdir pureftpd data
# docker run -d --restart unless-stopped --name pure-ftpd  -e MIN_PASV_PORT=40000 -e MAX_PASV_PORT=40009 -p 21:21  -p 40000-40009:40000-40009  -v $(pwd)/pureftpd:/etc/pureftpd  -v $(pwd)/data:/home/ftpuser gists/pure-ftpd:arm64
```
运行以下命令配置vsftpd的权限，以及添加test用户，并刷新vsftpd本地配置文件: 

```
 docker exec -it pure-ftpd chown ftpuser:ftpuser -R /home/ftpuser
 docker exec -it pure-ftpd pure-pw useradd test -m -u ftpuser -d /home/ftpuser/test
 docker exec -it pure-ftpd pure-pw mkdb
```
### 客户端
举winscp ftp连接为例,  新建一个ftp连接:      

![./images/2020_09_07_10_25_02_615x286.jpg](./images/2020_09_07_10_25_02_615x286.jpg)

直接在winscp里拖拉实现上传下载:    

![./images/2020_09_07_10_26_36_990x327.jpg](./images/2020_09_07_10_26_36_990x327.jpg)


