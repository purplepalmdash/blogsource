---
categories: ["web"]
comments: true
date: 2015-01-22T00:00:00Z
title: Enable Apache2 Redirect
url: /2015/01/22/enable-apache2-redirect/
---

### Problem
Want to redirect from `http://xxx/ ` to `http://xxx/a/b`    
### Solution
Change the configuration file of the `/etc/apache2/sites-enabled/000-default`, enable the  `RedirectMatch`:    

```
	# For forwarding all of the request to '/' TO '/bin/view'
 	RedirectMatch ^/$ /a/b

```
Restart the service of apache2 then everything goes OK.    
