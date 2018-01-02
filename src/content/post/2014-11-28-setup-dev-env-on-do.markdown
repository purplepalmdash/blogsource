---
categories: ["Linux"]
comments: true
date: 2014-11-28T00:00:00Z
title: Setup Dev Env On DO
url: /2014/11/28/setup-dev-env-on-do/
---

### Prepare
Install the following packages:   

```
$ sudo apt-get install python-virtualenv
$ sudo apt-get install ruby-full ruby
$ sudo gem install rhc

```
Since DO's network is pretty good, so it's very swift for developing on it.    
### TextBlob

```
$ virtualenv venv --python=python2.7
$ . venv/bin/activate
$ pip install textblob
$ python -m textblob.download_corpora
$ pip install flask


```
