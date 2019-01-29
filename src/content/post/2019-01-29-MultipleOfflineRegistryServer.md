+++
title = "MultipleOfflineRegistryServer"
date = "2019-01-29T16:09:15+08:00"
description = "MultipleOfflineRegistryServer"
keywords = ["Linux"]
categories = ["Linux"]
+++
Suppose your kubernetes clusters are in totally offline environment, you don't
have any connection to Internet, how to deal with the `docker pull` request
within the cluster? Following are steps for solving this problem.    

### Docker-Composed Registry
This step is pretty simple, I mainly refers to:    

[https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04)    

Following the steps in article, I got a running docker-compose triggered
docker registry server.    

Following is my folder structure:    

```
# tree .
├── data
├── docker-compose.yml
├── nginx
│   ├── openssl.cnf
│   ├── registry.conf
│   ├── registry.password
│   ├── server.crt
│   ├── server.csr
│   └── server.key
```

Docker-compose file is listed as:     

```
# cat docker-compose.yml 
nginx:
  image: "nginx:1.9"
  ports:
    - 443:443
  links:
    - registry:registry
  volumes:
    - ./nginx/:/etc/nginx/conf.d:ro

registry:
  image: registry:2
  ports:
    - 127.0.0.1:5000:5000
  environment:
    REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
  volumes:
    - ./data:/data
```
### Handle SSL
The above steps will only create a single hostname ssl enabled docker registry
server, we want to enable multi-domain self-signed SSL certificate.      

Create a file called openssl.cnf with the following details, notice that we
have set `docker.io`, `gcr.io`, `quay.io` and `elastic.co`.

```
# vim openssl.cnf
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req

[req_distinguished_name]
countryName = SL
countryName_default = SL
stateOrProvinceName = Western
stateOrProvinceName_default = Western
localityName = Colombo
localityName_default = Colombo
organizationalUnitName = ABC
organizationalUnitName_default = ABC
commonName = *.docker.io
commonName_max = 64

[ v3_req ]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = *.docker.io
DNS.2 = *.gcr.io
DNS.3 = *.quay.io
DNS.4 = *.elastic.co
DNS.5 = docker.io
DNS.6 = gcr.io
DNS.7 = quay.io
DNS.8 = elastic.co
```

Create the Private key.    

```
# rm -f server.key 
# sudo openssl genrsa -out server.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.....................+++++
......................................+++++
e is 65537 (0x010001)
```

Create Certificate Signing Request (CSR).    

```
# rm -f server.csr 
# sudo openssl req -new -out server.csr -key server.key -config openssl.cnf
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
SL [SL]:
Western [Western]:
Colombo [Colombo]:
ABC [ABC]:
*.docker.io []:
```
`*.docker.io` is from the above defined parts in `openssl.cnf`.    

Sign the SSL Certificate.     

```
# rm -f server.crt 
# sudo openssl x509 -req -days 36500 -in server.csr -signkey server.key -out server.crt -extensions v3_req -extfile openssl.cnf
Signature ok
subject=C = SL, ST = Western, L = Colombo, OU = ABC
Getting Private key
```
You can check the ssl certificated content via:    

```
# openssl x509 -in server.crt -noout -text 
Certificate:
    Data:
....

        Validity
            Not Before: Jan 29 08:30:20 2019 GMT
            Not After : Jan  5 08:30:20 2119 GMT
....

            X509v3 Subject Alternative Name: 
                DNS:*.docker.io, DNS:*.gcr.io, DNS:*.quay.io, DNS:*.elastic.co, DNS:docker.io, DNS:gcr.io, DNS:quay.io, DNS:elastic.co
```

### nginx conf
Add the domain items into registry.conf:    

```
upstream docker-registry {
  server registry:5000;
}

server {
  listen 443;
  server_name docker.io gcr.io quay.io elastic.co;

  # SSL
  ssl on;
  ssl_certificate /etc/nginx/conf.d/server.crt;
  ssl_certificate_key /etc/nginx/conf.d/server.key;
```

### dns configuration
Install bind9 on ubuntu, or install dnsmasq on centos system. take bind9's
configuration for example, add following configurations:    

```
# ls db*
db.docker.io  db.gcr.io  db.quay.io
```
The `named.conf.default-zones` is listed as following:    

```
# cat named.conf.default-zones 
// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/etc/bind/db.root";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};

zone "docker.io" {
        type master;
        file "/etc/bind/db.docker.io"; 
};

zone "quay.io" {
        type master;
        file "/etc/bind/db.quay.io"; 
};

zone "gcr.io" {
        type master;
        file "/etc/bind/db.gcr.io"; 
};

zone "elastic.co" {
        type master;
        file "/etc/bind/db.elastic.co"; 
};
```
Take `elastic.co` for example, show its content:     

```
# cat db.elastic.co
$TTL    604800
@       IN      SOA     elastic.co. root.localhost. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
@       IN      A      192.168.122.154
elastic.co IN      NS      192.168.122.154

docker     IN      A       192.168.122.154
```

### Using the docker registry
Point the nodes' dns server to your registry nodes, so if you ping elastic.co,
you got reply from `192.168.122.154`.     


Then add the crt into your docker's trusted sites:    


```
# cp server.crt /usr/local/share/ca-certificates
# update-ca-certificates
# systemctl restart docker
# docker login -u dddd -p gggg docker.io
# docker login -u dddd -p gggg quay.io
# docker login -u dddd -p gggg gcr.io
# docker login -u dddd -p gggg k8s.gcr.io
```
Now you can freely push/pull images from offline registry server, as if you
are on Internet.    
