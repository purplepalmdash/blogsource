+++
categories = ["Technology"]
date = "2020-05-28T11:22:49+08:00"
description = "WorkingTipsOnGreenMonitor"
keywords = ["Technology"]
title= "WorkingTipsOnGreenMonitor"

+++
### Netdata
Netdata 二进制 download:    

[https://github.com/netdata/netdata/releases](https://github.com/netdata/netdata/releases)     

选择 ` netdata-v1.22.1.gz.run `, 安装:    

```
# chmod netdata-v1.22.1.gz.run
# ./netdata-v1.22.1.gz.run --accept
# chkconfig netdata on
```

### node_exporter
二进制 download:    

[https://github.com/prometheus/node_exporter/releases](https://github.com/prometheus/node_exporter/releases)    

选择 `node_exporter-1.0.0.linux-amd64.tar.gz`  安装:     

```
#  tar xzvf node_exporter-1.0.0.linux-amd64.tar.gz
# cp node_exporter-1.0.0.linux-amd64/node_exporter  /usr/bin && chmod 777 /usr/bin/node_exporter
# vim /etc/rc.local
/usr/bin/node_exporter &
# /usr/bin/node_exporter  &
```

### Result
Netdata:    

![/images/2020_05_28_11_34_37_717x396.jpg](/images/2020_05_28_11_34_37_717x396.jpg)

node_exporter:    

![/images/2020_05_28_11_35_07_715x499.jpg](/images/2020_05_28_11_35_07_715x499.jpg)


