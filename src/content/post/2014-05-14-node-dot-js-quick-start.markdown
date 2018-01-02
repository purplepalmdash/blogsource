---
categories: ["nodejs"]
comments: true
date: 2014-05-14T00:00:00Z
title: Node.js Quick Start
url: /2014/05/14/node-dot-js-quick-start/
---

### Installation
Via following command

```
$ yaourt -S nodejs

```
### Quick Start

```
$ node
> console.log("Hello!")
Hello!
undefined

```
Hit twice "Ctrl+c" you will get out of the terminal.     
A simple example is like:   

```
var http = require("http");

http.createServer(function(request, response) {
	  response.writeHead(200, {"Content-Type": "text/plain"});
	    response.write("Hello World");
	      response.end();
}).listen(8888);
$ node server.js

$ curl http://localhost:8888
Hello World%

```

