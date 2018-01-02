---
categories: ["git"]
comments: true
date: 2014-10-14T00:00:00Z
title: Verified to github
url: /2014/10/14/verified-to-github/
---

### General Method
When you were denied by github, use following method for get through:    
Generate the key-gen

```
ssh-keygen

```
Then Creat the following files:    

```
# cat /home/Trusty/.ssh/config
Host github.com
User xxx@xxx.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443

```
Now use following command for setting the connection:    

```
ssh -T git@github.com

```
### Under firewall
First add the ~/.ssh/id_rsa.pub to the github's "SSH keys".     
If your machine runs under the firewall, you have to do things like following:     

```
ssh -L 2121:github.com:22 root@1xxx.xxx.xxx.158

```
Then in another terminal, run:   

```
ssh -T git@localhost -p 2121
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.

```
