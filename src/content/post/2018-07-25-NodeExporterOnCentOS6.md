+++
title = "NodeExporterOnCentOS6"
date = "2018-07-25T17:17:30+08:00"
description = "NodeExporterOnCentOS6"
keywords = ["Linux"]
categories = ["Linux"]
+++
从
[https://github.com/prometheus/node_exporter/releases](https://github.com/prometheus/node_exporter/releases),
下载适合机器架构的`node_exporter`版本，服务器上，通常下载为`amd64`的tar.gz包，而后解压。    

下载daemonize的rpm包安装，我们需要这个包来保证`node_exporter`的daemon运行.    

```
# wget https://forensics.cert.org/centos/cert/6/x86_64//daemonize-1.7.3-7.el6.x86_64.rpm
# rpm -ivh daemonize-1.7.3-7.el6.x86_64.rpm
```
解压`node_exporter`至`/usr/bin`目录下，现在开始撰写初始化脚本。    

```
# vim /etc/sysconfig/node_exporter

    #!/bin/bash
    #
    #	/etc/rc.d/init.d/node_exporter
    #
    # chkconfig: 2345 80 80
    #
    # config: /etc/prometheus/node_exporter.conf
    # pidfile: /var/run/prometheus/node_exporter.pid
    
    # Source function library.
    . /etc/init.d/functions
    
    
    RETVAL=0
    PROG="node_exporter"
    DAEMON_SYSCONFIG=/etc/sysconfig/${PROG}
    DAEMON=/usr/bin/${PROG}
    PID_FILE=/var/run/prometheus/${PROG}.pid
    LOCK_FILE=/var/lock/subsys/${PROG}
    LOG_FILE=/var/log/prometheus/node_exporter.log
    DAEMON_USER="prometheus"
    FQDN=$(hostname --long)
    GOMAXPROCS=$(grep -c ^processor /proc/cpuinfo)
    
    . ${DAEMON_SYSCONFIG}
    
    start() {
      if check_status > /dev/null; then
        echo "node_exporter is already running"
        exit 0
      fi
    
      echo -n $"Starting node_exporter: "
      daemonize -u ${DAEMON_USER} -p ${PID_FILE} -l ${LOCK_FILE} -a -e ${LOG_FILE} -o ${LOG_FILE} ${DAEMON} ${ARGS}
      RETVAL=$?
      echo ""
      return $RETVAL
    }
    
    stop() {
        echo -n $"Stopping node_exporter: "
        killproc -p ${PID_FILE} -d 10 ${DAEMON}
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && rm -f ${LOCK_FILE} ${PID_FILE}
        return $RETVAL
    }  
    
    check_status() {
        status -p ${PID_FILE} ${DAEMON}
        RETVAL=$?
        return $RETVAL
    }
    
    case "$1" in
        start)
            start
            ;;
        stop)
            stop
            ;;
        status)
    	check_status
            ;;
        reload|force-reload)
            reload
            ;;
        restart)
            stop
            start
            ;;
        *)
            N=/etc/init.d/${NAME}
            echo "Usage: $N {start|stop|status|restart|force-reload}" >&2
            RETVAL=2
            ;;
    esac
    
    exit ${RETVAL}
```
同时在`/etc/sysconfig`目录下创建`node_exporter`文件，默认的我们不添加任何初始化参数（后面你可以添加自定义的启动参数用于控制node_exporter的启动）:    

```
# vim /etc/sysconfig/node_exporter
ARGS=""
```
我们的初始化脚本默认使用了`prometheus`用户，因而我们需要在系统中手动创建该用户，并创建对应的目录，以承载pid文件及存放log等:    

```
# groupadd -r prometheus
# mkdir -p /usr/hostonnet/prometheus/
# useradd -r -g prometheus -s /sbin/nologin -d /usr/hostonnet/prometheus/ -c "prometheus Daemons" prometheus
# chown -R prometheus:prometheus /usr/hostonnet/prometheus/
# chown prometheus:prometheus /var/log/prometheus.log
# mkdir -p /var/run/prometheus/
# mkdir -p /var/log/prometheus/
# touch /var/log/prometheus/node_exporter.log
# chmod 777 /var/log/prometheus/node_exporter.log 
# chown prometheus:prometheus /var/log/prometheus/node_exporter.log
# /etc/init.d/node_exporter start
# chkconfig node_exporter on
```

检查，打开浏览器，查看`http://ipaddress:9100/metrics`是否可以看到对应导出的metrics即可。    
prometheus的配置文件中，添加上该节点的metrics导出信息即可。    
