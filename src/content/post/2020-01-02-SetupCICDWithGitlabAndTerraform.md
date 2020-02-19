+++
title = "SetupCICDWithGitlabAndTerraform"
date = "2020-01-02T10:43:35+08:00"
description = "SetupCICDWithGitlabAndTerraform"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Gitlab
Setup gitlab using docker-compose from:     

[https://github.com/sameersbn/docker-gitlab](https://github.com/sameersbn/docker-gitlab)    

Changes to the `docker-compose.yml`:     

```
    volumes:
    - ./redis-data:/var/lib/redis:Z
    volumes:
    - ./postgresql-data:/var/lib/postgresql:Z
    ports:
    - "10080:80"
    - "10022:22"
    volumes:
    - ./gitlab-data:/home/git/data:Z
```
Write a systemd file for controlling the docker-composed gitlab:     

```
# vim  /etc/systemd/system/gitlab.service 
[Unit]
Description=gitlab
Requires=docker.service
After=docker.service

[Service]
WorkingDirectory=/media/sdb/gitlab
Type=idle
Restart=always
# Remove old container items
ExecStartPre=/usr/bin/docker-compose -f /media/sdb/gitlab/docker-compose.yml down
# Compose up
ExecStart=/usr/bin/docker-compose -f /media/sdb/gitlab/docker-compose.yml up
# Compose stop
ExecStop=/usr/bin/docker-compose -f /media/sdb/gitlab/docker-compose.yml stop

[Install]
WantedBy=multi-user.target
# systemctl enable gitlab && systemctl start gitlab
```
### gitlab-runner
Install gitlab-runner via binary:     

```
# wget https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
# mv gitlab-runner-linux-amd64 /usr/bin/gitlab-runner-linux
# chmod 777 /usr/bin/gitlab-runner-linux
# sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
# mkdir -p /media/sdb/gitlab-runner
# chmod 777 -R /media/sdb/gitlab-runner
# sudo gitlab-runner install --user=root --working-directory=/media/sdb/gitlab-runner
# sudo gitlab-runner start
```
gitlab-runner register with the gitlab configuration.    
### Call terraform
With following files and configurations:     

```
# ls main.tf cloud_init.cfg
cloud_init.cfg  main.tf
# cat .gitlab-ci.yml
stages:
  - build
  - deploy

build-full-packages:
  stage: build
  # the tag 'shell' advices only GitLab runners using this tag to pick up that job
  tags:
    - xxxxxxcitag
  script:
    - date>time.txt
    - whoami
    - wget -q -O staticfiles.zip http://xxxxxx:10388/s/8MXHiafPeWABsB7/download
    - ./extract.sh
    - rm -f staticfiles.zip
    - rm -rf Rong_Kubesphere_Static/
    - which terraform
    - terraform init
    - terraform plan
    - terraform apply -auto-approve
    - ansible-playbook -i  /etc/ansible/terraform.py 0_preinstall/init.yml -b --flush-cache
    - ansible-playbook -i  /etc/ansible/terraform.py 1_k8s/cluster.yml --extra-vars @kiking-vars.yml --flush-cache
    - ansible-playbook -i  /etc/ansible/terraform.py 2_addons/addons.yml --extra-vars @kiking-vars.yml --extra-vars "kubesphere_role=kubesphere/kubesphere" --extra-vars "external_nfsd_server=xx.xx.xxx.166" --extra-vars "external_nfsd_path=/media/md0/nfs/tftmp"  --flush-cache
    - date>>time.txt
```
Thus you could run the terraform enabled environment for gitlab ci/cd
