+++
title= "UsingRedashForVisualization"
date = "2020-03-13T19:56:52+08:00"
description = "UsingRedashForVisualization"
keywords = ["Technology"]
categories = ["Technology"]
+++
### AIM
Using redash for representing Wuhan's nCov dataset.   

### Environment
Hardware/Software/OS is listed as following:     

```
kvm vm with 2cores and 3072MB memory plus 200G disk.
Using docker/docker-compose for redash website.
OS is ubuntu 18.04.3
IP: 192.168.137.100
```

### Redash bootstrap
Using the repository which located at:     
[https://github.com/getredash/setup](https://github.com/getredash/setup)    

You have to git clone this repository and run setup.sh under the directory. After setup the docker-compose file located in `/opt/redash`, if you want to quickly setup another docker/docker-compose based environment, simply copy the folder and the related docker images to remote machine's `/opt/redash`, up and running again.     

```
root@node:/opt/redash# ls
docker-compose.yml  env  postgres-data
root@node:/opt/redash# docker-compose run --rm server create_db
root@node:/opt/redash# docker-compose up -d
Creating network "redash_default" with the default driver
Creating redash_redis_1    ... done
Creating redash_postgres_1 ... done
Creating redash_scheduled_worker_1 ... done
Creating redash_adhoc_worker_1     ... done
Creating redash_scheduler_1        ... done
Creating redash_server_1           ... done
Creating redash_nginx_1            ... done
```
The redash environment rested in 7 docker instance, could be examined using `docker ps`:     

```
root@node:/opt/redash# docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                         NAMES
1e701e0bb8d2        redash/nginx:latest          "nginx -g 'daemon of…"   6 minutes ago       Up 6 minutes        0.0.0.0:80->80/tcp, 443/tcp   redash_nginx_1
5c00c609baa6        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        0.0.0.0:5000->5000/tcp        redash_server_1
8dcd4c2db0f8        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        5000/tcp                      redash_scheduled_worker_1
5d1dbd77f4ce        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        5000/tcp                      redash_adhoc_worker_1
6d56d7cd9040        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        5000/tcp                      redash_scheduler_1
c8efb3ff7437        postgres:9.6-alpine          "docker-entrypoint.s…"   7 minutes ago       Up 7 minutes        5432/tcp                      redash_postgres_1
a78dd1b1727f        redis:5.0-alpine             "docker-entrypoint.s…"   7 minutes ago       Up 7 minutes        6379/tcp                      redash_redis_1
```
Now visit the vm's 80 port you will see:    

![/images/2020_03_13_22_52_24_544x553.jpg](/images/2020_03_13_22_52_24_544x553.jpg)

After login you will get the following page:     

![/images/2020_03_13_22_55_16_827x528.jpg](/images/2020_03_13_22_55_16_827x528.jpg)

Until now your redash environment has been bootstraped.    

### Configure data source
Click your username and select `Data Sources`:    

![/images/2020_03_13_22_55_46_255x410.jpg](/images/2020_03_13_22_55_46_255x410.jpg)

Click `New Data Source` button for adding a new data source:    

![/images/2020_03_13_22_58_02_812x505.jpg](/images/2020_03_13_22_58_02_812x505.jpg)

Select sqlite and you will be leading to following window:     

![/images/2020_03_13_23_00_27_509x445.jpg](/images/2020_03_13_23_00_27_509x445.jpg)

Fill in some infos for finishing:    

![/images/2020_03_13_23_00_53_405x372.jpg](/images/2020_03_13_23_00_53_405x372.jpg)

### Get Sqlite db
We take following page for refrence:     

[https://discuss.redash.io/t/example-data-source-for-the-choropleth-maps/3696](https://discuss.redash.io/t/example-data-source-for-the-choropleth-maps/3696)    

First fetch the example sqlite db using following command:    

![/images/2020_03_13_23_03_22_565x368.jpg](/images/2020_03_13_23_03_22_565x368.jpg)

```
# wget https://dbhub.io/x/download/justinclift/DB4S%20daily%20users%20by%20country.sqlite?commit=28a554c2795170d5739b7a41df9baa2ad13b985b325d238bd869a14d1148f9ea
# mv DB4S\ daily\ users\ by\ country.sqlite first.db
```
Copy this db into following docker instance:    

```
# docker cp first.db redash_scheduled_worker_1:/app
# docker cp first.db redash_server_1:/app
# docker cp first.db redash_adhoc_worker_1:/app
# docker cp first.db redash_scheduler_1:/app
```

Now click `Test Connection` you will get `Succeed`.   
### Create Query
Click `Create Query`, you will be leading to following window:     

![/images/2020_03_13_23_10_34_613x380.jpg](/images/2020_03_13_23_10_34_613x380.jpg)

Create a new query:    

```
SELECT country, users
FROM query_6
WHERE date = '2019-04-22'
```

![/images/2020_03_13_23_13_35_681x275.jpg](/images/2020_03_13_23_13_35_681x275.jpg)

Click `Execute` you will get the table listed as:    

![/images/2020_03_13_23_13_59_1157x317.jpg](/images/2020_03_13_23_13_59_1157x317.jpg)

Click `New Visualization` for editing the visualization:     

![/images/2020_03_13_23_14_47_955x617.jpg](/images/2020_03_13_23_14_47_955x617.jpg)

Edit like following:    

![/images/2020_03_13_23_17_03_634x623.jpg](/images/2020_03_13_23_17_03_634x623.jpg)

The map will be rendered like:    

![/images/2020_03_13_23_17_39_922x579.jpg](/images/2020_03_13_23_17_39_922x579.jpg)

Click `save` for saving the map, then rename this query:     

![/images/2020_03_13_23_18_36_559x261.jpg](/images/2020_03_13_23_18_36_559x261.jpg)

### Dashboard
Create the first dashboard:     

![/images/2020_03_13_23_19_59_566x223.jpg](/images/2020_03_13_23_19_59_566x223.jpg)

An empty dashboard:     

![/images/2020_03_13_23_20_20_753x511.jpg](/images/2020_03_13_23_20_20_753x511.jpg)

Click `Add Widget`, select our newly created map:    

![/images/2020_03_13_23_20_39_676x251.jpg](/images/2020_03_13_23_20_39_676x251.jpg)

Select the map:     

![/images/2020_03_13_23_21_16_691x280.jpg](/images/2020_03_13_23_21_16_691x280.jpg)

Effect:    

![/images/2020_03_13_23_21_33_921x659.jpg](/images/2020_03_13_23_21_33_921x659.jpg)

### Conclusion
Now we have created the first visualization in redash easily displayed a
sqlite database, in next chapter we will take a look at how to display the
nCov statistics.   
