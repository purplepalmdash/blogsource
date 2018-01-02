+++
title = "WorkingTipsOnRPIAndRedSocks"
date = "2017-12-24T22:10:31+08:00"
description = "WorkingtipsonRPIAndRedSocks"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
To setup an wireless ap which could enable all of the equipment free to
internet.    

### Startup
Download the rpi image from:    
[http://downloads.raspberrypi.org/raspbian/images/raspbian-2017-12-01/2017-11-29-raspbian-stretch.zip](http://downloads.raspberrypi.org/raspbian/images/raspbian-2017-12-01/2017-11-29-raspbian-stretch.zip)    
Then unzip this image file and write to sd card.    

Plugin your sd card into your rpi board, startup. At the very beginning, you
have to enable the ssh via:    

```
$ sudo systemctl enable ssh && sudo systemctl start ssh
```
Then you could login remotely using ssh, default username/password is
`pi/raspberry`.    

Edit the sources.list and install some necessary packages:    

```
$ sudo vim /etc/apt/sources.list
deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main non-free contrib
$ sudo apt-get update && sudo apt-get install -y vim 
```
Enable the `ip_forward` via:    

```
$ sudo vim /etc/sysctl.conf
net.ipv4.ip_forward=1
$ sudo systcl -p 
```

### Network Configuration
Configure the network via:    

```
$  cat /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)

# Please note that this file is written to be used with dhcpcd
# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

##### To use hostapd you need do following.
auto wlan0
iface wlan0 inet static  
    address 10.0.70.1
        netmask 255.255.255.0
auto eth0
iface eth0 inet static  
    address 192.168.0.15
        netmask 255.255.255.0
        gateway 192.168.0.1
```
Restart your network you will get the wlan0 as `10.0.70.1`, while your wired
network acts as the `192.168.0.15` in ethernet.     

### dnsmasq and pdnsd
Configure the dnsmasq as the dhcpd server via:    

```
# apt-get install dnsmasq
# vim /etc/dnsmasq.conf
port=9053
#配置监听地址
listen-address=127.0.0.1,10.0.70.1
#配置DHCP分配段
dhcp-range=10.0.70.50,10.0.70.150,12h
dhcp-option=6,192.168.0.15
```
While your dns was polluted via the fucking gfw, you have to use pdnsd for
getting the correct dns.    

```
# apt-get install pdnsd
# vim /etc/pdnsd.conf
```
My pdnsd configuration file is listed as following:    

```
// Read the pdnsd.conf(5) manpage for an explanation of the options.

/* Note: this file is overriden by automatic config files when
  /etc/default/pdnsd AUTO_MODE is set and that
  /usr/share/pdnsd/pdnsd-$AUTO_MODE.conf exists
*/

global {
    perm_cache=4096;
    cache_dir="/var/cache/pdnsd";
    run_as="pdnsd";
    server_ip = 192.168.0.15;
    server_port=53;
    status_ctl = on;
    query_method=tcp_only;
    neg_domain_pol = off;  
    paranoid = on;  
    par_queries = 1;  
    min_ttl = 6h;  
    max_ttl = 12h;  
    timeout = 10;  
}

/* with status_ctl=on and resolvconf installed, this will work out from the box
  this is the recommended setup for mobile machines */
server {  
   label = "routine";
   ip = 223.5.5.5;
   timeout = 5;  
   reject = 74.125.127.102,
       74.125.155.102,  
       74.125.39.102,  
       74.125.39.113,  
       209.85.229.138,  
       128.121.126.139,  
       159.106.121.75,  
       169.132.13.103,  
       192.67.198.6,  
       202.106.1.2,  
       202.181.7.85,  
       203.161.230.171,  
       203.98.7.65,  
       207.12.88.98,  
       208.56.31.43,  
       209.145.54.50,  
       209.220.30.174,  
       209.36.73.33,  
       211.94.66.147,  
       213.169.251.35,  
       216.221.188.182,  
       216.234.179.13,  
       243.185.187.39,  
       37.61.54.158,  
       4.36.66.178,  
       46.82.174.68,  
       59.24.3.173,  
       64.33.88.161,  
       64.33.99.47,  
       64.66.163.251,  
       65.104.202.252,  
       65.160.219.113,  
       66.45.252.237,  
       69.55.52.253,  
       72.14.205.104,  
       72.14.205.99,  
       78.16.49.15,  
       8.7.198.45,  
       93.46.8.89,  
       37.61.54.158,  
       243.185.187.39,  
       190.93.247.4,  
       190.93.246.4,  
       190.93.245.4,  
       190.93.244.4,  
       65.49.2.178,  
       189.163.17.5,  
       23.89.5.60,  
       49.2.123.56,  
       54.76.135.1,  
       77.4.7.92,  
       118.5.49.6,  
       159.24.3.173,  
       188.5.4.96,  
       197.4.4.12,  
       220.250.64.24,  
       243.185.187.30,  
       249.129.46.48,  
       253.157.14.165;  
   reject_policy = fail;  
   exclude = ".google.com",  
       ".cn",
       ".baidu.com",
       ".qq.com",
       ".gstatic.com",  
       ".googleusercontent.com",  
       ".googlepages.com",  
       ".googlevideo.com",  
       ".googlecode.com",  
       ".googleapis.com",  
       ".googlesource.com",  
       ".googledrive.com",  
       ".ggpht.com",  
       ".youtube.com",  
       ".youtu.be",  
       ".ytimg.com",  
       ".twitter.com",  
       ".facebook.com",  
       ".fastly.net",  
       ".akamai.net",  
       ".akamaiedge.net",  
       ".akamaihd.net",  
       ".edgesuite.net",  
       ".edgekey.net";  
}

server {  
   # Better setup dns server(DON'T USE PORT 53) on your own vps for faster proxying  
   label = "special";
   ip = 208.67.222.222,208.67.220.220;
   port = 5353;
   proxy_only = on;  
   timeout = 5;  
}  
# Generated by resolvconf
server {
	label=resolvconf;
	ip=192.168.0.15;
}
# End of resolvconf
```
Enable the auto-start of pdnsd:    

```
$ sudo vim /etc/default/pdnsd
START_DAEMON=yes
$ sudo systemctl enable pdnsd
$ sudo systemctl starrt pdnsd
```
Examine via:    

```
$ sudo dig @192.168.0.15 www.youtube.com
```

### ShadowSocks and RedSock2
Install shadowsocks via:    

```
$ sudo pip install shadowsocks
$ sudo vim /etc/shadowsocks.conf
$ sudo vim /etc/rc.local
sslocal -c /etc/shadowsocks.conf -d start
```
Trouble-Shooting On sslocal:    

```
$ vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
Comment all of the :    

libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx)
Changes to : libcrypto.EVP_CIPHER_CTX_reset(self._ctx)
# rm -f  /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.pyc
```


Redsocks:    

```
$ git clone https://github.com/semigodking/redsocks
$ cd redsocks
$ sudo apt-get install libevent-dev libssl-dev libpolarssl-dev
$ make USE_CRYPTO_POLARSSL=true
```
Configure the redsocks via:    

```
$ vim redsocks.conf
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
	log = stderr;
	// log = "file:/path/to/file";
	// log = "syslog:local7";

	// detach from console
	daemon = on;

	/* Change uid, gid and root directory, these options require root
	 * privilegies on startup.
	 * Note, your chroot may requre /etc/localtime if you write log to syslog.
	 * Log is opened before chroot & uid changing.
	 * Debian, Ubuntu and some other distributions use `nogroup` instead of
	 * `nobody`, so change it according to your system if you want redsocks
	 * to drop root privileges.
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

	/* Override per-socket values for TCP_KEEPIDLE, TCP_KEEPCNT,
	 * and TCP_KEEPINTVL. see man 7 tcp for details.
	 * `redsocks' relies on SO_KEEPALIVE option heavily. */
	//tcp_keepalive_time = 0;
	//tcp_keepalive_probes = 0;
	//tcp_keepalive_intvl = 0;
}

redsocks {
	/* `local_ip' defaults to 127.0.0.1 for security reasons,
	 * use 0.0.0.0 if you want to listen on every interface.
	 * `local_*' are used as port to redirect to.
	 */
	local_ip = 0.0.0.0;
	local_port = 12345;

	// listen() queue length. Default value is SOMAXCONN and it should be
	// good enough for most of us.
	listenq = 128; // SOMAXCONN equals 128 on my Linux box.

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
	// The two fields are meaningless when proxy type is 'direct'.
	ip = 127.0.0.1;
	port = 1080;

	// known types: socks4, socks5, http-connect, http-relay
	// New types: direct, shadowsocks, https-connect
	type = socks5;

	// Specify interface for outgoing connections.
	// This is useful when you have multiple connections to
	// internet or when you have VPN connections.
	// interface = tun0;

	// Change this parameter to 1 if you want auto proxy feature. 
	// When autoproxy is set to non-zero, the connection to target
	// will be made directly first. If direct connection to target
	// fails for timeout/connection refuse, redsocks will try to
	// connect to target via the proxy.
	autoproxy = 0;
	// timeout is meaningful when 'autoproxy' is non-zero.
	// It specified timeout value when trying to connect to destination
	// directly. Default is 10 seconds. When it is set to 0, default
	// timeout value will be used.
	timeout = 10;

	// login = "foobar";// field 'login' is reused as encryption
					   // method of shadowsocks
	// password = "baz";
}

//*redudp {
//*	// `local_ip' should not be 0.0.0.0 as it's also used for outgoing
//*	// packets that are sent as replies - and it should be fixed
//*	// if we want NAT to work properly.
//*	local_ip = 127.0.0.1;
//*	local_port = 10053;
//*
//*	// `ip' and `port' of socks5 proxy server.
//*	ip = 10.0.0.1;
//*	port = 1080;
//*	login = username;// field 'login' is reused as encryption
//*					   // method of shadowsocks
//*	password = pazzw0rd;
//*
//*	// know types: socks5, shadowsocks
//*	type = socks5;
//*
//*	// redsocks knows about two options while redirecting UDP packets at
//*	// linux: TPROXY and REDIRECT.  TPROXY requires more complex routing
//*	// configuration and fresh kernel (>= 2.6.37 according to squid
//*	// developers[1]) but has hack-free way to get original destination
//*	// address, REDIRECT is easier to configure, but requires `dest_ip` and
//*	// `dest_port` to be set, limiting packet redirection to single
//*	// destination.
//*	// [1] http://wiki.squid-cache.org/Features/Tproxy4
//*	dest_ip = 8.8.8.8;
//*	dest_port = 53;
//*
//*	// Do not set it large if this section is for DNS requests. Otherwise,
//*	// you may encounter out of file descriptor problem. For DNS requests,
//*	// 10s is adequate.
//*	udp_timeout = 30;
//*	udp_timeout_stream = 180;
//*}
//*
//*tcpdns {
//*	// Transform UDP DNS requests into TCP DNS requests.
//*	// You can also redirect connections to external TCP DNS server to
//*	// REDSOCKS transparent proxy via iptables.
//*	//local_ip = 192.168.1.1; // Local server to act as DNS server
//*	//local_port = 1053;      // UDP port to receive UDP DNS requests
//*	//tcpdns1 = 8.8.4.4;      // DNS server that supports TCP DNS requests
//*	//tcpdns1_port = 53;      // DNS server port, default 53
//*	//tcpdns2 = 8.8.8.8;      // DNS server that supports TCP DNS requests
//*	//tcpdns2_port = 53;      // DNS server port, default 53
//*	//timeout = 4;            // Timeout value for TCP DNS requests
//*	local_ip=127.0.0.1;
//*	local_port=5300;
//*}

autoproxy {
    // Specify interface for outgoing connections.
    // This is useful when you have multiple connections to
    // internet or when you have VPN connections.
    // interface = wlan0;

    no_quick_check_seconds = 60; // Directly relay traffic to proxy if an IP
                                 // is found blocked in cache and it has been
                                 // added into cache no earlier than this
                                 // specified number of seconds.
                                 // Set it to 0 if you do not want to perform
                                 // quick check when an IP is found in blocked
                                 // IP cache, thus the connection will be
                                 // redirected to proxy immediately.
    quick_connect_timeout = 3;   // Timeout value when performing quick
                                 // connection check if an IP is found blocked
                                 // in cache.
}

ipcache {
    // Configure IP cache
    cache_size = 4;   // Maximum number of IP's in 1K. 
    stale_time = 900; // Seconds to stale an IP in cache since it is added
                      // into cahce.
                      // Set it to 0 to disable cache stale.
    port_check = 1;   // Whether to distinguish port number in address
    cache_file = "/tmp/ipcache.txt"; // File used to store blocked IP's in cache.
    autosave_interval = 3600; // Interval for saving ip cache into file.
                              // Set it to 0 to disable autosave.
    // When autosave_interval and stale_time are both 0, IP cache behaves like
    // a static blacklist. 
}

// you can add more `redsocks' and `redudp' sections if you need.
```
Now you could add following lines into your `/etc/rc.local` file to enable redsocks
 startup at the system boot:    

```
$ sudo vim /etc/rc.local
$ /home/pi/redsocks/redsocks2 -c /home/pi/redsocks/redsocks.conf
```
### hostapd
Setting hostapd for letting the wlan0 as an AP:   

```
$ sudo apt-get install -y hostapd
$ vim /etc/hostapd/hostapd.conf
interface=wlan0
hw_mode=g
channel=10
ieee80211d=1
country_code=CN
ieee80211n=1
wmm_enabled=1

ssid=sssremote
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase=xxxxxxxxxxx
$ sudo systemctl enable hostapd
$ sudo systemctl start hostapd
```
### iptables rules
Use iptables-persistent for saving the rules:    

```
$ sudo apt-get install -y iptables-persistent
$ sudo service netfilter-persistent save
$ sudo service netfilter-persistent rerload
```
Add following rules:    

```
$ sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
$ sudo iptables -t nat -N SHADOWSOCKS
$ sudo iptables -t nat -A SHADOWSOCKS -d $server_IP -j RETURN
$ sudo iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN  
$ sudo iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 12345
$ sudo iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS  
$ sudo iptables -t nat -A PREROUTING -p tcp -j SHADOWSOCKS  
```
Save the rules via:    

```
$ sudo service netfilter-persistent save
```

Add system-reboot rules:    

```
$ sudo vim /etc/rc.local
iptables-restore < /etc/iptables/rules.v4  
```
Now reboot your raspberryPi, you will get a free wifi ap.   
