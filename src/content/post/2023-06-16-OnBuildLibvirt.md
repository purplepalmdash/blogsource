+++
title= "OnBuildLibvirt"
date = "2023-06-16T15:34:54+08:00"
description = "OnBuildLibvirt"
keywords = ["Technology"]
categories = ["Technology"]
+++
System libs:    

```
libunistring-devel
```

gnutls `>=3.6.0` is required for building libvirtd, thus:    

```
[root@buildgen10 gnutls-3.6.0]# ./configure --prefix=/usr
  *** Libnettle 3.1 was not found.
```
Nettle:    

```
 wget https://ftp.gnu.org/gnu/nettle/nettle-3.1.1.tar.gz
 tar xzvf nettle-3.1.1.tar.gz 
 cd nettle-3.1.1
 ./configure --prefix=/usr
 make
 make install
```
Rebuild gnutls:    

```
./configure --prefix=/usr
make -j8
pkcs11_privkey.c:335:32: error: storage size of 'rsa_pss_params' isn't known
  struct ck_rsa_pkcs_pss_params rsa_pss_params;
```

https://www.frytea.com/archives/546/
