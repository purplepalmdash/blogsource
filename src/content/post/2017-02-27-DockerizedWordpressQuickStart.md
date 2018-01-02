+++
date = "2017-02-27T16:01:30+08:00"
categories = ["Linux"]
keywords = ["Linux"]
description = "Quick start wordpress on Docker"
title = "DockerizedWordpressQuickStart"

+++
### AIM
Recently I want to setup a wordpress based website and quickly adjust the
content, since the time is so limted, I choose docker for development and
finish the task soonly, this article records all of the detailed steps.    

### Docker-compose the wordpress
We use docker-compose for composing the wordpress based apps, the commands
listed as following:    

```
$ mkdir -p ~/code/wordpress
$ vim docker-compose.yml
```
The `docker-compose.yml` file is listed as following:    

```
version: '2'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: wordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     volumes:
       - html:/var/www/html
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
    html:
```
Now use following commands for startup the wordpress or delete all of the
content:   

```
#### Startup the website
$ sudo docker-compose up -d

#### Down the wordpress
$ sudo docker-compose down
```
Now pick up your browser for visting `http://YourIPAddress:8000` you could get
the wordpress installed and run.    

![/images/2017_02_27_16_31_18_400x737.jpg](/images/2017_02_27_16_31_18_400x737.jpg)    

![/images/2017_02_27_16_32_29_562x517.jpg](/images/2017_02_27_16_32_29_562x517.jpg)   
### Customize the website
We want to use the `makeclean` template for setting up our website(because I
am setting up a clean corporation website for my friends). Download it
directly from internet:    

```
$ wget http://dlw.press/themes/corporate/makeclean.zip
$ mkdir -p ~/code/wordpress/makeclean
$ cp makeclean.zip ~/code/wordpress/makeclean
$ cd ~/code/wordpress/makeclean && unzip makeclean.zip
```
Inspect the volumes and install the `makeclean` template:    

```
$ sudo  docker volume ls | grep html
local               makeclean_html
$ sudo docker volume inspect makeclean_html 
[
    {
        "Name": "makeclean_html",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/makeclean_html/_data",
        "Labels": null
    }
]
```
Now copy the `makeclean` template into the docker volume:    

```
# cp -r ~/code/wordpress/makeclean/ /var/lib/docker/volumes/makeclean_html/_data/wp-content/themes/
```
Now Open themes, you could see the `Make Clean` template has been installed:    

![/images/2017_02_27_16_34_01_924x426.jpg](/images/2017_02_27_16_34_01_924x426.jpg)    

Click `Enable` you could startup the template, then click `Begin installing
plugins` for installing the plugins:    

![/images/2017_02_27_16_35_11_473x313.jpg](/images/2017_02_27_16_35_11_473x313.jpg)    

Install the following plugins:    

![/images/2017_02_27_16_35_45_475x385.jpg](/images/2017_02_27_16_35_45_475x385.jpg)    

Press `One Click Demo Data` for enable the demo:    

![/images/2017_02_27_16_36_49_502x137.jpg](/images/2017_02_27_16_36_49_502x137.jpg)    

Click `02 Home2` and check all of the options and begin install, you should
take for a while for waiting all of the datas being downloaded into local
directory:    

![/images/2017_02_27_16_38_13_797x659.jpg](/images/2017_02_27_16_38_13_797x659.jpg)    

Be sure to activate the `Envato Toolkit` plugins.    

Now refresh the website's first page you will see the good-looking website has
been set.    

![/images/2017_02_27_16_41_36_1221x786.jpg](/images/2017_02_27_16_41_36_1221x786.jpg)    
