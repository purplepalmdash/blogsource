---

comments: true
date: 2013-10-21T00:00:00Z
title: snmp on Raspberry PI
url: /2013/10/21/snmp-on-raspberry-pi/
---

I want to use snmp for administrating my raspberry PI, for example, its disk usage, cpu usage, and etc. following is the how-to of setting up the monitoring environment.    
###Preparation

```
	$ apt-get upate && apt-get upgrade
	$ apt-get install apache2 php5 mysql-client mysql-server
You will be prompted to set a password for mysql root user.
	$ apt-get install php5-mysql php5-snmp rrdtool snmp snmpd
```

