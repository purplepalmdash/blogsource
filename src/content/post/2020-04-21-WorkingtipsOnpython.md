+++
categories = ["Technology"]
date = "2020-04-21T17:46:31+08:00"
description = "WorkingtipsOnpython"
keywords = ["Technology"]
title= "WorkingtipsOnpython"

+++
Just recording:    

```
[root@3652a460ae13 apps]# python manage.py shell
Python 3.6.1 (default, Jun 29 2018, 02:56:19)                          
Type 'copyright', 'credits' or 'license' for more information
IPython 6.5.0 -- An enhanced Interactive Python. Type '?' for help. 
                                                      
In [1]: import requests
   ...: import time
   ...: 
   ...: from kubeops_api.apps_client import AppsClient
   ...: from kubeops_api.models.host import Host
   ...: from kubeops_api.cluster_data import LokiContainer
   ...: 
   ...: 

In [2]: import kubernetes.client
   ...: import redis
   ...: import json
   ...: import logging
   ...: import kubeoperator.settings
   ...: import log.es
   ...: import datetime, time
   ...: import builtins
   ...: 
   ...: from kubernetes.client.rest import ApiException
   ...: from kubeops_api.cluster_data import ClusterData, Pod, NameSpace, Node, Container, Deployment, StorageClass, PVC, Event
   ...: from kubeops_api.models.cluster import Cluster
   ...: from kubeops_api.prometheus_client import PrometheusClient
   ...: from kubeops_api.models.host import Host
   ...: from django.db.models import Q
   ...: from kubeops_api.cluster_health_data import ClusterHealthData
   ...: from django.utils import timezone
   ...: from ansible_api.models.inventory import Host as C_Host
   ...: from common.ssh import SSHClient, SshConfig
   ...: from message_center.message_client import MessageClient
   ...: from kubeops_api.utils.date_encoder import DateEncoder
   ...: 
   ...: 

In [3]: 

In [3]: project_name = "kingston"

In [4]: cluster = Cluster.objects.get(name=project_name)

In [5]: host = "loki.apps.kingston.mydomain.com"

In [6]: config = {
   ...:   'host': host,
   ...:   'cluster': cluster
   ...: }

In [7]: 

In [7]: print(config)
{'host': 'loki.apps.kingston.mydomain.com', 'cluster': <Cluster: kingston>}

In [8]: prom_client = PrometheusClient(config)

In [9]: label_url = "http://loki.apps.kingston.mydomain.com/loki/api/v1/label/container_name/values"

In [10]: app_client = AppsClient(cluster=cluster)

In [11]:  label_query_url = label_url.format(host="loki.apps.kingston.mydomain.com")

In [12]: label_req = app_client.get('loki', label_query_url)

In [13]: label_req.ok
Out[13]: True

In [14]: now = time.time()

In [15]: end = int(round(now * 1000 * 1000000))

In [16]: start = int(round(now * 1000 - 3600000) * 1000000)

In [17]: label_req_json = label_req.json()

In [18]: print(label_req_json)
{'status': 'success', 'data': ['autoscaler', 'calico-kube-controllers', 'calico-node', 'chartmuseum', 'chartsvc', 'controller', 'coredns', 'dashboard', 'grafana', 'install-cni', 'kube-apiserver', 'kube-controller-manager', 'kube-proxy', 'kube-scheduler', 'kubeapps-plus-mongodb', 'kubernetes-dashboard', 'loki', 'metrics-server', 'metrics-server-nanny', 'nfs-client-provisioner', 'nginx', 'node-problem-detector', 'prometheus-alertmanager', 'prometheus-alertmanager-configmap-reload', 'prometheus-kube-state-metrics', 'prometheus-node-exporter', 'prometheus-server', 'prometheus-server-configmap-reload', 'promtail', 'proxy', 'registry', 'registry-ui', 'sync', 'tiller', 'traefik-ingress-lb']}

In [19]: values = label_req_json.get('data', [])

In [20]: print(values)
['autoscaler', 'calico-kube-controllers', 'calico-node', 'chartmuseum', 'chartsvc', 'controller', 'coredns', 'dashboard', 'grafana', 'install-cni', 'kube-apiserver', 'kube-controller-manager', 'kube-proxy', 'kube-scheduler', 'kubeapps-plus-mongodb', 'kubernetes-dashboard', 'loki', 'metrics-server', 'metrics-server-nanny', 'nfs-client-provisioner', 'nginx', 'node-problem-detector', 'prometheus-alertmanager', 'prometheus-alertmanager-configmap-reload', 'prometheus-kube-state-metrics', 'prometheus-node-exporter', 'prometheus-server', 'prometheus-server-configmap-reload', 'promtail', 'proxy', 'registry', 'registry-ui', 'sync', 'tiller', 'traefik-ingress-lb']

In [21]: for name in values:
    ...:     error_count = 0
    ...:     prom_url = 'http://{host}/api/prom/query?limit=1000&query={{container_name="{name}"}}&start={start}&end={end}'
    ...:     prom_query_url = prom_url.format(host="loki.apps.kingston.mydomain.com", name=name, start=start, end=end)
    ...:     prom_req = app_client.get('loki', prom_query_url)
    ...:     if prom_req.ok:
    ...:         prom_req_json = prom_req.json()
    ...:         streams = prom_req_json.get('streams', [])
    ...:         for stream in streams:
    ...:             entries = stream.get('entries', [])
    ...:             for entry in entries:
    ...:                 line = entry.get('line', None)
    ...:                 print(line)


In [29]: for name in values:                                                                                                             
    ...:     error_count = 0                                   
    ...:     prom_url = 'http://{host}/api/prom/query?limit=1000&query={{container_name="{name}"}}&start={start}&end={end}'
    ...:     prom_query_url = prom_url.format(host="loki.apps.kingston.mydomain.com", name=name, start=start, end=end)
    ...:     prom_req = app_client.get('loki', prom_query_url)
    ...:     if prom_req.ok:
    ...:         prom_req_json = prom_req.json()
    ...:         streams = prom_req_json.get('streams', [])
    ...:         for stream in streams:
    ...:             entries = stream.get('entries', [])
    ...:             for entry in entries:
    ...:                 line = entry.get('line', None)
    ...:                 if line is not None and 'level=error' in line:
    ...:                     error_count = error_count + 1

```
Thus you could fetch the correct python logs for kubeoperator
