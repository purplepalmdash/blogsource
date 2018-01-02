+++
date = "2017-03-02T14:22:31+08:00"
description = "Migration DockerCompose based wordpress"
title = "Migration Docker Based WordPress"
categories = ["Linux"]
keywords = ["Linux"]

+++
### AIM
For last 3 days I was busy designing the website for a company, I run
everything in docker, and finish the design very fast. Now I want to migrate
it from the old vps to a new vps, following records all of the steps.    

### Data Volume Migration
Examine the data volume via following commands:    

```
# docker volume ls | grep db
local               makeclean_db_data
# docker volume ls | grep html
local               makeclean_html
```
Get the volume mount point:    

```
# docker volume inspect makeclean_html | grep -i mountpoint
        "Mountpoint": "/var/lib/docker/volumes/makeclean_html/_data",
# docker volume inspect makeclean_db_data | grep -i mountpoint
        "Mountpoint": "/var/lib/docker/volumes/makeclean_db_data/_data",
```
Now zip all of the data volume:   

```
# tar czf db_data.tar.gz /var/lib/docker/volumes/makeclean_db_data/_data
# tar czf html.tar.gz /var/lib/docker/volumes/makeclean_html/_data
```
Copy the 2 gz files to your new vps.   

### Adjust
Start the docker-compose like the steps in last article, then manually stop
the docker instance, replace the data volume using the 2 gz files, start the
docker instance again.    

Now you will be leading the old domain name, we need to change the options of
wordpress database, we create a new mysql docker instance for changing the
wordpress database:    

```
# docker run -it --link wordpress_db_1:mysql --net wordpress_default mysql:5.7 /bin/bash
root@95f633180461:/#
```
In this newly created docker instance you could use mysql client tools for
changing wordpress options(`172.19.0.2` is your mysql server):     

```
# mysql -h172.19.0.2 -P3306 -uroot -pwordpress
```
Mysql adjust commands ls listed as following:     

```
use wordpress;
UPDATE wp_options SET option_value = replace(option_value, 'xxx.xxx.222.222:8000','localhost:8000') ;
UPDATE wp_posts SET post_content = replace(post_content, 'xxx.xxx.222.222:8000','localhost:8000') ;
UPDATE wp_comments SET comment_content = replace(comment_content, 'xxx.xxx.222.222:8000', 'localhost:8000') ;
UPDATE wp_comments SET comment_author_url = replace(comment_author_url, 'xxx.xxx.222.222:8000', 'localhost:8000') ;
\q
```
Now restart the docker instance, you will get the wordpress running in new
vps.     

