+++
title = "GrafanaTemplateIssue"
date = "2017-10-17T17:06:46+08:00"
description = "GrafanaTemplateIssue"
keywords = ["Linux"]
categories = ["Linux"]
+++
I downloaded some template from grafana.net, but none of them work properly,
following are the trouble-shooting of template.    

1. Change the data source.    
View the json file, get its input field:    

```
{
  "__inputs": [
    {
      "name": "DS_BC-GRAPHITE",
      "label": "BC-Graphite",
      "description": "",
      "type": "datasource",
      "pluginId": "graphite",
      "pluginName": "Graphite"
    }
  ],

```
This means you have to define a datasource named `DS_BC-GRAPHITE`, like
following:    

![/images/2017_10_17_17_09_04_681x396.jpg](/images/2017_10_17_17_09_04_681x396.jpg)

2. Change collectd's write_graphite structure.    

![/images/2017_10_17_17_09_29_304x241.jpg](/images/2017_10_17_17_09_29_304x241.jpg)

The item could not be displayed properly, because the definition for data
listed as:    

![/images/2017_10_17_17_10_20_490x287.jpg](/images/2017_10_17_17_10_20_490x287.jpg)


so you should have following definition in `/etc/collectd.conf`:    

![/images/2017_10_17_17_10_58_469x424.jpg](/images/2017_10_17_17_10_58_469x424.jpg)

Notice `collectd.`, this means you could use `collectd.*` for getting the
data.    
