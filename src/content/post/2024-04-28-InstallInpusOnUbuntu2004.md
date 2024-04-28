+++
title= "InstallInpusOnUbuntu2004"
date = "2024-04-28T09:31:37+08:00"
description = "InstallInpusOnUbuntu2004"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. Install/Configuration
Import keyring:     

```
mkdir -p /etc/apt/keyrings/
curl -fsSL https://pkgs.zabbly.com/key.asc -o /etc/apt/keyrings/zabbly.asc
```
update repository and install inpus:     

```
apt update -y
apt install -y incus
 apt install -y incus-ui-canonical
incus config set core.https_address :8443
```
Generate cert file in browser:     

![/images/2024_04_28_09_32_56_1083x755.jpg](/images/2024_04_28_09_32_56_1083x755.jpg)

In incus server:     

```
root@hope:~# cp /home/dash/Downloads/incus-ui.crt ./Downloads/
root@hope:~# incus config trust add-certificate Downloads/incus-ui.crt
```

In browser:    

![/images/2024_04_28_09_34_37_1035x384.jpg](/images/2024_04_28_09_34_37_1035x384.jpg)

The import file should be:    

![/images/2024_04_28_09_34_57_414x98.jpg](/images/2024_04_28_09_34_57_414x98.jpg)

Then back to browser windows, confirm the imported cert:   

![/images/2024_04_28_09_35_08_630x331.jpg](/images/2024_04_28_09_35_08_630x331.jpg)

Your UI would be looks like:    

![/images/2024_04_28_09_35_46_945x645.jpg](/images/2024_04_28_09_35_46_945x645.jpg)

Add your user into incus group:    

```
sudo adduser dash incus-admin
```
Init the incus:      

```
$ incus admin init
Would you like to use clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (dir, lvm, lvmcluster, btrfs) [default=btrfs]: dir
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=incusbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
Would you like the server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]: no
Would you like a YAML "init" preseed to be printed? (yes/no) [default=no]: 
```
Create the first instance:      

```
incus launch images:ubuntu/22.04 first
```
Finally we could remove lxd:     

```
$ sudo snap remove lxd
```

### 2. images
list image:   

```
$ incus image list
+-------+--------------+--------+----------------------------------------+--------------+-----------+-----------+----------------------+
| ALIAS | FINGERPRINT  | PUBLIC |              DESCRIPTION               | ARCHITECTURE |   TYPE    |   SIZE    |     UPLOAD DATE      |
+-------+--------------+--------+----------------------------------------+--------------+-----------+-----------+----------------------+
|       | 8b2691953577 | no     | Debian bookworm amd64 (20240424_05:24) | x86_64       | CONTAINER | 94.50MiB  | 2024/04/28 02:09 UTC |
+-------+--------------+--------+----------------------------------------+--------------+-----------+-----------+----------------------+
|       | 479d8812eada | no     | Ubuntu jammy amd64 (20240427_07:42)    | x86_64       | CONTAINER | 120.93MiB | 2024/04/28 01:55 UTC |
+-------+--------------+--------+----------------------------------------+--------------+-----------+-----------+----------------------+

```

incus remote show images repositories:   

```
$ incus remote list
+-----------------+------------------------------------+---------------+-------------+--------+--------+--------+
|      NAME       |                URL                 |   PROTOCOL    |  AUTH TYPE  | PUBLIC | STATIC | GLOBAL |
+-----------------+------------------------------------+---------------+-------------+--------+--------+--------+
| images          | https://images.linuxcontainers.org | simplestreams | none        | YES    | NO     | NO     |
+-----------------+------------------------------------+---------------+-------------+--------+--------+--------+
| local (current) | unix://                            | incus         | file access | NO     | YES    | NO     |
+-----------------+------------------------------------+---------------+-------------+--------+--------+--------+

```

search images:   

```
$ incus image list images: bookworm
+--------------------------------+--------------+--------+----------------------------------------+--------------+-----------------+-----------+----------------------+
|             ALIAS              | FINGERPRINT  | PUBLIC |              DESCRIPTION               | ARCHITECTURE |      TYPE       |   SIZE    |     UPLOAD DATE      |
+--------------------------------+--------------+--------+----------------------------------------+--------------+-----------------+-----------+----------------------+
| debian/12 (7 more)             | 2b5e45154f58 | yes    | Debian bookworm amd64 (20240424_05:24) | x86_64       | VIRTUAL-MACHINE | 349.13MiB | 2024/04/24 00:00 UTC |
+--------------------------------+--------------+--------+----------------------------------------+--------------+-----------------+-----------+----------------------+
| debian/12 (7 more)             | 8b2691953577 | yes    | Debian bookworm amd64 (20240424_05:24) | x86_64       | CONTAINER       | 94.50MiB  | 2024/04/24 00:00 UTC |
+--------------------------------+--------------+--------+----------------------------------------+--------------+-----------------+-----------+----------------------+
| debian/12/arm64 (3 more)       | dbba0a514259 | yes    | Debian bookworm arm64 (20240424_05:24) | aarch64      | CONTAINER       | 91.50MiB  | 2024/04/24 00:00 UTC |
+--------------------------------+--------------+--------+----------------------------------------+--------------+-----------------+-----------+----------------------+
| debian/12/arm64 (3 more)       | e2fc3247a569 | yes    | Debian bookworm arm64 (20240424_05:24) | aarch64      | VIRTUAL-MACHINE | 338.21MiB | 2024/04/24 00:00 UTC |
+--------------------------------+--------------+--------+----------------------------------------+--------------+-----------------+-----------+----------------------+
....
```

launch:     

```
incus launch -p default -p bridgeprofile images:debian/12 kissdebian
```
