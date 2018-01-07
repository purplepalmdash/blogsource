+++
title = "pdnsForArchLinux"
date = "2018-01-05T10:54:05+08:00"
description = "pdnsForArchLinux"
keywords = ["Linux"]
categories = ["Linux"]
+++
For avoiding dns resolving pollution, I need to configure pdnsd for my
archlinux workstation, following are the steps:

```
# pacman -S pdnsd
# cp /usr/share/doc/pdnsd/pdnsd.conf /etc
# vim /etc/pdnsd.conf
```
The configuration file is listed as:    

```
global {
	perm_cache=10240;
	cache_dir="/var/cache/pdnsd";
#	pid_file = /var/run/pdnsd.pid;
	run_as="pdnsd";
	server_ip = 127.0.0.1;  # Use eth0 here if you want to allow other
				# machines on your network to query pdnsd.
	server_port=53;
	status_ctl = on;
#	paranoid=on;       # This option reduces the chance of cache poisoning
	                   # but may make pdnsd less efficient, unfortunately.
	query_method=tcp_only;
	#min_ttl=15m;       # Retain cached entries at least 15 minutes.
	#max_ttl=1w;        # One week.
	#timeout=10;        # Global timeout option (10 seconds).
	#neg_domain_pol=on;
	#udpbufsize=1024;   # Upper limit on the size of UDP messages.
    neg_domain_pol = off;    
    paranoid = on;    
    par_queries = 1;    
    min_ttl = 1d;    
    max_ttl = 5d;    
    timeout = 10; 
}

# The following section is most appropriate if you have a fixed connection to
# the Internet and an ISP which provides good DNS servers.
server {
	label= "routine";
	ip = 223.5.5.5;  # Put your ISP's DNS-server address(es) here.
#	proxy_only=on;     # Do not query any name servers beside your ISP's.
	                   # This may be necessary if you are behind some
	                   # kind of firewall and cannot receive replies
	                   # from outside name servers.
	timeout=5;         # Server timeout; this may be much shorter
			   # that the global timeout option.
#	uptest=if;         # Test if the network interface is active.
#	interface=eth0;    # The name of the interface to check.
#	interval=10m;      # Check every 10 minutes.
#	purge_cache=off;   # Keep stale cache entries in case the ISP's
#			   # DNS servers go offline.
#	edns_query=yes;    # Use EDNS for outgoing queries to allow UDP messages
#			   # larger than 512 bytes. May cause trouble with some
#			   # legacy systems.
#	exclude=.thepiratebay.org,  # If your ISP censors certain names, you may
#		.thepiratebay.se,   # want to exclude them here, and provide an
#		.piratebay.org,	    # alternative server section below that will
#		.piratebay.se;	    # successfully resolve the names.
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
```
Then you have to enable and start the pdnsd service via:    

```
# systemctl enable pdnsd
# systemctl start pdnsd
# vim /etc/resolv.con
nameserver 127.0.0.1
# chattr +i /etc/resolv.conf
```
you could use dig for testing your pdnsd configuration.     
