+++
title= "OnMeshCommander"
date = "2023-08-04T08:57:11+08:00"
description = "OnMeshCommander"
keywords = ["Technology"]
categories = ["Technology"]
+++
Steps:    

```
# docker run -it ubuntu:latest /bin/bash
In docker instance:    
# cd /root
# mkdir meshcommander
# cd meshcommander
# npm install meshcommander 
```
Then commit this docker instance to docker image:     

```
# docker commit awgowugowugouawo meshcommander:mine
```
Start the instance:     

```
docker run -it -p 4080:3000 meshcommander:ctyun /bin/bash
```

Edit the node modules:    

```
root@938cedc55cde:~/meshcommander# vim node_modules/meshcommander/webserver.js 
156         obj.app.listen(port, '0.0.0.0', function () { console.log("MeshCommander running on http://127.0.0.1:" + port + '.'); });
node node_modules/meshcommander/
```
Visit the `https://ip:4080` you could reache the meshcommander webpage.    

![/images/2023_08_04_09_31_09_640x215.jpg](/images/2023_08_04_09_31_09_640x215.jpg)

Configuration:    

![/images/2023_08_04_09_31_33_433x288.jpg](/images/2023_08_04_09_31_33_433x288.jpg)

System Status:     

![./images/2023_08_04_09_32_08_363x424.jpg](./images/2023_08_04_09_32_08_363x424.jpg)

Attach an iso for Linux installation：    


meshcommander is too old. switch to 
