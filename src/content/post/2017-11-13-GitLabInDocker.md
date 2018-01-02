+++
title = "GitLabInDocker"
date = "2017-11-13T10:44:15+08:00"
description = "GitLabDeployment in docker"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
Run the docker instance via following steps:    

```
# docker pull gitlab/gitlab-ce
# mkdir -p ~/gitlab
# mkdir -p ~/gitlab/config
# mkdir -p ~/gitlab/logs
# mkdir -p ~/gitlab/data
# docker run --detach --publish 443:443 --publish 80:80 --publish 2222:22 \ 
--name gitlab \
--memory 4g \
--restart always \
--volume ~/gitlab/config:/etc/gitlab \
--volume ~/gitlab/logs:/var/log/gitlab \
--volume ~/gitlab/data:/var/opt/gitlab \ 
gitlab/gitlab-ce:latest
```

Reconfigure the configuration files:    

```
# vim ~/gitlab/config/gitlab.rb
external_url 'http://192.192.189.129'
```

Let the configuration take effect:    

```
# docker exec -it xxxxx /bin/bash
# gitlab-ctl reconfigure
```

Now open your browser and visit `http://192.192.189.129` then you could access
your gitlab website.
