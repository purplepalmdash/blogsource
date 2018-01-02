---
categories: ["virtualization"]
comments: true
date: 2015-06-03T21:00:20Z
title: Tips on using vagrant and chefdk
url: /2015/06/03/tips-on-using-vagrant-and-chefdk/
---

1. You should install all of the gem of `berkshelf` via:     

```
$ gem install berkshelf
$ /opt/chef/embedded/bin/gem install berkshelf
$ /opt/vagrant/embedded/bin/gem install berkshelf
```

2. Besure to add following into your PATH:    

```
$  echo $PATH
/opt/chefdk/bin:/home/kkk/.rvm/gems/ruby-2.2.1/bin:/home/kkk/.rvm/gems/ruby-2.2.1@global/bin:/home/kkk/.rvm/rubies/ruby-2.2.1/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/home/kkk/.rvm/bin:/home/kkk/.rvm/bin:/home/kkk/.rvm/bin
```

So now you could continue with `vagrant up` or other steps.    
