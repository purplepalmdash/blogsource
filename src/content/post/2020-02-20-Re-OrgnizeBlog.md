+++
title = "Re Organize My Blog Structure"
date = "2020-02-20T08:24:41+08:00"
description = "re-orgnize-my-blog-structure"
keywords = ["Technology"]
categories = ["Technology"]
+++
Recently I find it's necessary to re-orgnize my blog structure, for past 8
years I've written nearly 1000 articles in this blog, they covered so many
technologies, from embedded system to cloud-computing, also with my life's
blog. So I simply use `Technology` and `Life` for classifying them. Also I
have to adjust my websiste's compatibility to newest hugo(v0.64.0), previously
I use an old version(v0.31.0) for building the whole website. Following
are the steps for doing such a complicated task.    

### 1. Structure Re-Orgnization

Replace all of the `md` files and `markdown` files's `categories`:    

```
$ cd /home/xxxxx/Code/purplepalmxxxx.github.io/src/content/post
$ find . | xargs -I % sed -i 's/categories\ =.*/categories\ =\ ["Technology"]/g' %
$ find . | xargs -I % sed -i 's/categories:.*/categories:\ ["Technology"]/g' %
```

Some of the old markdown files didn't have categories, manually add them:     

```
$ sed -i '2s/$/categories:\ ["Technology"]/'  2013-07-*
$ sed -i '2s/$/categories:\ ["Technology"]/'  2013-08-*
$ sed -i '2s/$/categories:\ ["Technology"]/'  2013-09-*
$ sed -i '2s/$/categories:\ ["Technology"]/'  2013-10-*
$ sed -i '2s/$/categories:\ ["Technology"]/'  2013-11-0*
$ sed -i '2s/$/categories:\ ["Technology"]/'  2013-11-11*
$ sed -i '2s/$/categories:\ ["Technology"]/'  2013-11-12*
```

By-now all of the posts are `Technology` related, manually change some posts
to `LinuxTips` and `Life`.    

### 2. Upgrade hugo
Download the newest hugo version from official site and put it in binary
directory:     

```
$ cd binaries 
$ ls
hugo  hugo_v031
$ ./hugo version
Hugo Static Site Generator v0.64.1/extended linux/amd64 BuildDate: unknown
```

Adjust some for generate properly(hyde-a theme adjustment):     

```
$ rm -f
/home/xxxx/Code/purplepalmxxxx.github.io/src/themes/hyde-a/layouts/post/post.html
$ vim
/home/xxxx/Code/purplepalmxxxx.github.io/src/themes/hyde-a/layouts/index.html 
{{ partial "head.html" . }}
<div class="content container">
  <div class="posts">
+++    {{ $paginator := .Paginate (where .Site.RegularPages "Type" "in" site.Params.mainSections) }}
    {{ range $paginator.Pages }}
```
Now commit all of the changes, the ci/cd for blogging will automatically use
the newest version of hugo for building-out the static website.      
