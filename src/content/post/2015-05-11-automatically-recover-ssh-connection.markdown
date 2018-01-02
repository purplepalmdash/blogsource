---
categories: ["Linux"]
comments: true
date: 2015-05-11T00:00:00Z
title: Automatically Recover SSH Connection
url: /2015/05/11/automatically-recover-ssh-connection/
---

Thanks for the Great File Wall, my ssh connection to my vps is not stable, so I use following scripts for automatically maintain the ssh conneciton, once the connection down, it will restart immediately.    

```
$ vim autokeepssh.sh 
#!/bin/bash

while [ '' == '' ]
do
        # Use ssh -R for reverse ssh
        ssh_d_process_num=`ps aux|grep -E 'ssh -NfR' |grep -v grep |wc -l`
        if [ "$ssh_d_process_num" == "0" ]; then
                # Automatically start the ssh proxy 
                echo "Autostart!"
                ssh -NfR 4389:localhost:22 Trusty@xxx.xxx.xxx.xxx -p xxxx &
        #else
        #       echo 'ssh -d running'
        fi

        sleep 5
done

```
-R means I started a reverse connection.    
