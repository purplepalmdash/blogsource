---
categories: ["null"]
comments: true
date: 2014-04-15T00:00:00Z
title: Miragate Wordpress From Development Machine To Server
url: /2014/04/15/miragate-wordpress-from-development-machine-to-server/
---

First Log on to the development machine, view the configuration file of the Wordpress:    

```
	// ** MySQL settings - You can get this info from your web host ** //
	/** The name of the database for WordPress */
	define('DB_NAME', 'wordpress');
	
	/** MySQL database username */
	define('DB_USER', 'wordpressuser');
	
	/** MySQL database password */
	define('DB_PASSWORD', 'password');
	
	/** MySQL hostname */
	define('DB_HOST', 'localhost');

```
Dump out the database:    
	mysqldump --user=wordpressuser -p wordpress >wordpress.sql
Copy the dev database into the server's directory, then view the configure of the server's wordpress configuration.    

```
	// ** MySQL settings - You can get this info from your web host ** //
	/** The name of the database for WordPress */
	define('DB_NAME', 'wordpress');
	
	/** MySQL database username */
	define('DB_USER', 'wordpressuser');
	
	/** MySQL database password */
	define('DB_PASSWORD', 'password');
	
	/** MySQL hostname */
	define('DB_HOST', 'localhost');

```
They are the same, then begin dump into server's:     

```
	mysqldump --user=wordpressuser -p wordpress < ./wordpress.sql

```

Using plugins for database migration, 
plugins, search: wp migrate db
Install the same theme, called "BlackBird"    


Database drop first: 

```
	mysqladmin -uroot -pxxxxxxx  drop wordpress

```
Then import the new database via: 


via phpmyadmin:
Install phpmyadmin: 

failed


Copy the directory from dev machine to server machine, unzip it and replace the /srv/http/wordpress. 

```
	sudo pacman -S phpmyadmin php-mcrypt

```

To be continued. 
