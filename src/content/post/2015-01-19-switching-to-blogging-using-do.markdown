---
categories: ["web"]
comments: true
date: 2015-01-19T00:00:00Z
title: Switching To Blogging Using DO
url: /2015/01/19/switching-to-blogging-using-do/
---

Since I changed the working PC, so I need to switch from the old machine to the new machine, while the new machine's network speed is pretty slow, that force me to switching from the local side working to vps-side working, following is the steps for Using DigitalOcean for updating my blog.     
### Repository
First using the git for pulling the repository from the github, following this article for setting the whole octopress system:    
[http://kkkttt.github.io/blog/2014/07/30/zai-githubshang-bu-shu-ni-de-octopressbo-ke/](http://kkkttt.github.io/blog/2014/07/30/zai-githubshang-bu-shu-ni-de-octopressbo-ke/)    
### New Tips
For the `_deploy` directory is the newly generated one, we have to do following changes:    

```
rake setup_github_pages
cd _deploy
rm -f ./index.html
git remove index.html
git pull origin master
cd ..
rake generate && rake deply

```
After the above steps, Writing blog in Octopress becomes possible.    
