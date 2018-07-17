+++
title = "GitLabOnUbuntu"
date = "2018-07-17T09:44:20+08:00"
description = "GitLabOnUbuntu"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Install
Install via:    

```
# apt-get install postfix gitlab-ce gitlab-runner
```
### Configure
Configure the `external_url`:    

```
# vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.122.252'
# gitlab-ctl reconfigure && gitlab-ctl restart
```

Configure the gitlab-ci-multi-runner, take care of the gitlab-ci token is
taken from the webUI:    

```
# gitlab-ci-multi-runner register
```

![/images/2018_07_17_11_03_52_811x360.jpg](/images/2018_07_17_11_03_52_811x360.jpg)

Setup the runner for specified project:    

![/images/2018_07_17_11_44_39_463x459.jpg](/images/2018_07_17_11_44_39_463x459.jpg)

Configuration for the runner:    

![/images/2018_07_17_11_48_57_759x379.jpg](/images/2018_07_17_11_48_57_759x379.jpg)

Change the gitlab-runner as root

```
# vim /etc/systemd/system/gitlab-runner.service
Change from --user gitlab-runner to --user root
# systemctl daemon-reload
# systemctl restart gitlab-runner.service
```
