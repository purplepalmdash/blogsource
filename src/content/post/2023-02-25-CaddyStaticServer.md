+++
title= "CaddyStaticServer"
date = "2023-02-25T22:50:24+08:00"
description = "CaddyStaticServer"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install canddy on rpi via:     

```
# sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https -y
# apt-cache search caddy
# curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
# sudo apt update
# sudo apt-get install caddy -y
```
Configuration files:    

```
$ cat /etc/caddy/Caddyfile 
# The Caddyfile is an easy way to configure your Caddy web server.
#
# Unless the file starts with a global options block, the first
# uncommented line is always the address of your site.
#
# To use your own domain name (with automatic HTTPS), first make
# sure your domain's A/AAAA DNS records are properly pointed to
# this machine's public IP, then replace ":80" below with your
# domain name.

:80 {
	# Set this path to your site's directory.
	root * /media/sda/ucc
	# Enable the static file server.
	file_server browse {
	hide .git
}

basicauth * {
cwgo gowugowuoguwougowugouwogue
}
	# Another common task is to set up a reverse proxy:
	# reverse_proxy localhost:8080

	# Or serve a PHP site through php-fpm:
	# php_fastcgi localhost:9000
}

# Refer to the Caddy docs for more information:
# https://caddyserver.com/docs/caddyfile

```
The password could be generated via `caddy hash-password`.    

Reload the server:    

```
$  sudo systemctl reload caddy
```
