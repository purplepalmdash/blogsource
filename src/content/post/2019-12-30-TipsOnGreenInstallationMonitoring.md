+++
title = "TipsOnGreenInstallationMonitoring"
date = "2019-12-30T15:25:54+08:00"
description = "TipsOnGreenInstallationMonitoring"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
monitoring using green installation(prometheus and netdata)
### Netdata
Download the run.gz from official release:     

```
# curl https://github.com/netdata/netdata/releases/download/v1.19.0/netdata-v1.19.0.gz.run>netdata-v1.19.0.gz.run
# chmod 777 *.run
# ./netdata-v1.19.0.gz.run --accept
```
Todo:    

why could not be installed via:    `curl xxx/xxx.gz.run | bash` ?   

Tips: Pass parameter to bash:    

```
# curl xxxx/xxx.gz.run | bash -s -- --accept
```
### Prometheus
Install makeself via `apt-get install -y makeself`, later we will use it for createing install.run package.   

Folder structure:    

```
$ tree node_exporter 
node_exporter
├── install_node_exporter.sh
└── node_exporter
```
Edit the `install_node_exporter.sh` file:     

```
#!/bin/sh -e

_check_root () {
    if [ $(id -u) -ne 0 ]; then
        echo "Please run as root" >&2;
        exit 1;
    fi
}

_check_root

mkdir -p /opt/node_exporter
cp node_exporter /opt/node_exporter/

if [ -x "$(command -v systemctl)" ]; then
    cat << EOF > /lib/systemd/system/node-exporter.service
[Unit]
Description=Prometheus agent
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
ExecStart=/opt/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
EOF

    systemctl enable node-exporter
    systemctl start node-exporter
elif [ -x "$(command -v chckconfig)" ]; then
    cat << EOF >> /etc/inittab
::respawn:/opt/node_exporter/node_exporter
EOF
elif [ -x "$(command -v initctl)" ]; then
    cat << EOF > /etc/init/node-exporter.conf
start on runlevel [23456]
stop on runlevel [016]
exec /opt/node_exporter/node_exporter
respawn
EOF

    initctl reload-configuration
    stop node-exporter || true && start node-exporter
else
    echo "No known service management found" >&2;
    exit 1;
fi
```

While `node_exporter` is downloaded from github.    

Make install.run:    

```
# makeself ./node_exporter ./node_exporter_0.18.1.run "SFX installer for node_exporter(0.18.1)" ./install_node_exporter.sh
```

Thus we get the `run` file for installing:     

```
$ ls
node_exporter/  node_exporter_0.18.1.run
```
We can install it via `./node_exporter_0.18.1.run`.    
