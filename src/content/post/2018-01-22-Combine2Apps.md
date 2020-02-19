+++
title = "Combine2Apps"
date = "2018-01-22T16:50:23+08:00"
description = "Combine2Apps"
keywords = ["k8s"]
categories = ["Technology"]
+++
### AIM
Combine the tomcat and oracle db apps.    

tomcat -> war -> oracleDB.    

### Directory Stucture

![/images/2018_01_22_16_53_27_321x492.jpg](/images/2018_01_22_16_53_27_321x492.jpg)

`values.yaml` definition for oracle db:    

```
oracledb:
  enabled: true
```

requirements.yaml:    

```
requirements.yaml 
dependencies:
- name: oracledb
  version: 0.1.0
  repository: http://192.xxx.xxx.xxx/k8s/oracledb/
```

Charts.yaml, very simple:    

```
appVersion: 0.1.0
description: Chart for something
home: http://192.xxx.xxx.xxx/k8s/xxx
name: xxx
sources:
  - http://192.xxx.xxx.xxx/k8s/xxx
version: 0.1.0
```
configmap.yaml:    

```
jdbc.url=jdbc:oracle:thin:@{{ template "oracledb.fullname" . }}:1521/xe\r\njdbc.username=xxx\r\njdbc.password=xxx\r\n\r\n
```

oracledb definition:    

values.yaml

```
serviceType: ClusterIP
```

deployment.yaml:    

```
    spec:
      containers:
      - name: {{ template "fullname" . }}
        image: "{{ .Values.image }}"
        imagePullPolicy: {{ .Values.imagePullPolicy | quote }}
        ports:
        - name: oracle
          containerPort: 1521

```
svc.yaml:    

```
spec:
  type: {{ .Values.serviceType }}
  ports:
  - name: oracle
    port: 1521
    targetPort: oracle
  selector:
```
