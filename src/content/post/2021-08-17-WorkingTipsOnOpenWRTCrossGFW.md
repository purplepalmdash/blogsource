+++
title= "WorkingTipsOnOpenWRTCrossGFW"
date = "2021-08-17T13:55:59+08:00"
description = "WorkingTipsOnOpenWRTCrossGFW"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 硬件配置
2核2G, 虚拟机,
其磁盘文件为`openwrt-21.02.0-rc3-x86-64-generic-ext4-combined.img`.    

### 网络配置
隔离网络:     

![/images/2021_08_17_13_57_12_682x128.jpg](/images/2021_08_17_13_57_12_682x128.jpg)

NAT网络:    

![/images/2021_08_17_13_57_12_682x128.jpg](/images/2021_08_17_13_57_12_682x128.jpg)

启动完毕后，安装以下opkg包:    

```
# opkg install redsocks
```
OpenWRT中配置：    

Wan:    

![./images/2021_08_17_13_59_55_881x449.jpg](./images/2021_08_17_13_59_55_881x449.jpg)

![/images/2021_08_17_14_03_07_467x160.jpg](/images/2021_08_17_14_03_07_467x160.jpg)

LAN:

![/images/2021_08_17_14_00_17_604x551.jpg](/images/2021_08_17_14_00_17_604x551.jpg)

![/images/2021_08_17_14_02_14_508x152.jpg](/images/2021_08_17_14_02_14_508x152.jpg)

Enable dhcp server:    

![/images/2021_08_17_14_02_34_580x398.jpg](/images/2021_08_17_14_02_34_580x398.jpg)

Devices:    

![/images/2021_08_17_14_03_45_953x307.jpg](/images/2021_08_17_14_03_45_953x307.jpg)

![/images/2021_08_17_14_04_05_478x424.jpg](/images/2021_08_17_14_04_05_478x424.jpg)

![/images/2021_08_17_14_04_21_610x382.jpg](/images/2021_08_17_14_04_21_610x382.jpg)

DNS:    

![/images/2021_08_17_14_04_47_552x604.jpg](/images/2021_08_17_14_04_47_552x604.jpg)

### redsocks items
`/etc/redsocks.conf`:    

```
base {
        // debug: connection progress & client list on SIGUSR1
        log_debug = off;

        // info: start and end of client session
        log_info = on;

        /* possible `log' values are:
         *   stderr
         *   "file:/path/to/file"
         *   syslog:FACILITY  facility is any of "daemon", "local0"..."local7"
         */
        // log = stderr;
        // log = "file:/path/to/file";
        log = "syslog:local7";

        // detach from console
        daemon = on;

        /* Change uid, gid and root directory, these options require root
         * privilegies on startup.
         * Note, your chroot may requre /etc/localtime if you write log to syslog.
         * Log is opened before chroot & uid changing.
         */
        // user = nobody;
        // group = nobody;
        // chroot = "/var/chroot";

        /* possible `redirector' values are:
         *   iptables   - for Linux
         *   ipf        - for FreeBSD
         *   pf         - for OpenBSD
         *   generic    - some generic redirector that MAY work
         */
        redirector = iptables;
}

redsocks {
        /* `local_ip' defaults to 127.0.0.1 for security reasons,
         * use 0.0.0.0 if you want to listen on every interface.
         * `local_*' are used as port to redirect to.
         */
        local_ip = 192.168.89.254;
        local_port = 12345;

        // listen() queue length. Default value is SOMAXCONN and it should be
        // good enough for most of us.
        // listenq = 128; // SOMAXCONN equals 128 on my Linux box.

        // `max_accept_backoff` is a delay to retry `accept()` after accept
        // failure (e.g. due to lack of file descriptors). It's measured in
        // milliseconds and maximal value is 65535. `min_accept_backoff` is
        // used as initial backoff value and as a damper for `accept() after
        // close()` logic.
        // min_accept_backoff = 100;
        // max_accept_backoff = 60000;

        // `ip' and `port' are IP and tcp-port of proxy-server
        // You can also use hostname instead of IP, only one (random)
        // address of multihomed host will be used.
        ip = 10.50.208.147;
        port = 8118;


        // known types: socks4, socks5, http-connect, http-relay
        type = socks5;

        // login = "foobar";
        // password = "baz";
}

redudp {
        // `local_ip' should not be 0.0.0.0 as it's also used for outgoing
        // packets that are sent as replies - and it should be fixed
        // if we want NAT to work properly.
        local_ip = 127.0.0.1;
        local_port = 10053;
        // `ip' and `port' of socks5 proxy server.
        ip = 10.0.0.1;
       port = 1080;
        login = username;
        password = pazzw0rd;

        // redsocks knows about two options while redirecting UDP packets at
        // linux: TPROXY and REDIRECT.  TPROXY requires more complex routing
        // configuration and fresh kernel (>= 2.6.37 according to squid
        // developers[1]) but has hack-free way to get original destination
        // address, REDIRECT is easier to configure, but requires `dest_ip` and
        // `dest_port` to be set, limiting packet redirection to single
        // destination.
        // [1] http://wiki.squid-cache.org/Features/Tproxy4
        dest_ip = 8.8.8.8;
        dest_port = 53;

        udp_timeout = 30;
        udp_timeout_stream = 180;
}

dnstc {
        // fake and really dumb DNS server that returns "truncated answer" to
        // every query via UDP, RFC-compliant resolver should repeat same query
        // via TCP in this case.
        local_ip = 127.0.0.1;
        local_port = 5300;
}

// you can add more `redsocks' and `redudp' sections if you need.
```

`/etc/init.d/redsocks` content:    

```
##### /etc/init.d/redsocks######
#!/bin/sh /etc/rc.common
# Copyright (C) 2007 OpenWrt.org

START=90

# check if configuration exists
[ -e "/etc/redsocks.conf" ] || exit 0

start() {
        if [ -e "/var/run/redsocks.pid" ]; then
                echo "redsocks is already running"
                exit 0
        fi

        /bin/echo -n "running redsocks ..."

        # startup the safety-wrapper for the daemon
        /usr/sbin/redsocks -p /var/run/redsocks.pid

        /bin/echo " done"
}

stop() {
        if [ ! -e "/var/run/redsocks.pid" ]; then
                echo "redsocks is not running"
                exit 0
        fi

        /bin/echo -n "stopping redsocks ..."

        # kill the process
        /bin/kill $(cat /var/run/redsocks.pid)
        rm /var/run/redsocks.pid

        echo " done"
}

```

Iptables rules:    

```
iptables -t nat -N REDSOCKS
# Ignore LANs IP address
iptables -t nat -A REDSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A REDSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A REDSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A REDSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A REDSOCKS -d 240.0.0.0/4 -j RETURN
# Anything else should be redirected to redsocks's local port
iptables -t nat -A REDSOCKS -p tcp -j REDIRECT --to-ports 12345
iptables -t nat -I zone_lan_prerouting -j REDSOCKS
```

Then with a reverse proxy you could cross gfw.   
Reverse connect:    

Internet machine(with 53 as its dns server):    

```
ssh -o GatewayPorts=true -f -N -T -R *:18888:localhost:18888 docker@10.xxxxx
ssh -o GatewayPorts=true -f -N -T -R *:8118:10.10.3.19:1080 docker@10.xxxxxx
sudo socat tcp-listen:18888,reuseaddr,fork udp:127.0.0.1:53
```

Intranet machine:    

```
socat -T15 udp4-recvfrom:53,bind=10.xxx.xxx.xx,fork tcp:localhost:18888
```
