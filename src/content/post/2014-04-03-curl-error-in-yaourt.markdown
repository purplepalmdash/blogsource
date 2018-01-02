---
categories: ["ArchLinux"]
comments: true
date: 2014-04-03T00:00:00Z
title: CURL Error in Yaourt
url: /2014/04/03/curl-error-in-yaourt/
---

When downloading package, the yaourt complains:<br />

```
	yaourt curl: (60) SSL certificate problem

```
The solution is via:<br />
Add following lines at the top of the pkg build file:<br />

```
	DLAGENTS=("https::/usr/bin/curl -k -o %o %u")

```
Then restart the yaourt, you can pass through the building. 
