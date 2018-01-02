+++
date = "2017-06-30T08:50:26+08:00"
categories = ["Linux"]
keywords = ["WorkingTipsOnAnsiblePull"]
description = "WorkingTipsOnAnsiblePull"
title = "WorkingTipsOnAnsiblePull"

+++
### 背景
在生产环境中经常面临大量卸载或迁移服务器的操作，这时候必须手动重建配置，这是一项繁琐且容易出错的工作。    

部署人员可能会尝试DevOps方式("基础设施作为代码")管理底层设备，在迁移过程中也会尝试像Chef或Puppet这样的流行工具，但是发现它们对于简单的案例来说也是非常复杂的。    

本文将介绍一个强大但简单的配置管理工具Ansible, 基于Ansible和GitLab建立出的工作流，我们可以快速对环境进行集中化部署/管理。在实际的生产环境中，基于此工作流将使运维变得更加简单、高效、可靠。

### 概观
#### Ansible
Ansible是一个用Python编写的自动化和编排工具。它通过SSH连接工作，不需要在主机上安装代理。    

如果读者不了解使用公钥的无密码连接，可以参考:    

Ansible的配置文件以YAML文档编写，称为"Playbook", Playbook中可以包括用户自定义的任务和事件处理程序。举例说，在一个用于配置服务器的Playbook中，我们定义一系列用于更新配置文件的数据库服务器，而事件处理程序则在这些任务完成之后负责重新启动数据库服务。    

#### 基于推送的Ansible
Ansible用于推送的架构:    

![/images/2017_06_30_09_12_05_632x643.jpg](/images/2017_06_30_09_12_05_632x643.jpg)

Control Host: 通常是用于管理的节点(本机)，在主机上手动运行需要运行的任务，也可以基于crontab定时来做。    
Ansible使用`inventory`文件来包含需配置的机器列表，在上图中，我们包含了两个组:
数据库服务器组和Web服务器组。`inventory`文件中包含了这些服务器组中的节点IP。    
`inventory`文件也可以是动态生成的，Ansible同样提供了对动态生成的支持。    
每个Playbook从inventory文件中选取一个或多个组进行配置。在上一个例子中，我们定义了一个Playbook配置和编排数据库服务器，另一个则作用于Web服务器。Playbook也可以被抽象成更高层次，因此可以将两者中比较共同的任务配置成一个作用于all节点的任务。    

#### 基于拉取的Ansible
使用`ansible-pull`命令可以开启Ansible的拉取工作方式，ansible-pull命令的执行流程如下:   

1. 每个主机都安装了Ansible，
2. 该配置存储在Git仓库中，
3. ansible-pull检出配置库在给定的分支或标记（提示：prod，staging等），
4. ansible-pull 执行指定的playbook，
5. 使用cronjob自动化ansible-pull进程
6. 最后，针对每次配置的修改，您所要做的就是将配置更改推送到Git仓库。

![/images/2017_06_30_09_24_08_676x633.jpg](/images/2017_06_30_09_24_08_676x633.jpg)

#### GitLab
GitLab是一个利用 Ruby on Rails 开发的开源应用程序，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。    

GitLab可以与诸多CI(持续集成)工具协同工作，且提供了原生的GitLabCI用于持续CI和CD。    

GitLab提供完整的单机/集群解决方案。

### 工作流讲解
#### 环境配置
本工作流示例环境包括如下机器:    

192.168.33.2, GitLab VM, 2Core 2G.    
192.168.33.10,  mgmt VM, 256 M内存。    
192.168.33.11,  lb VM, 256 M内存。    
192.168.33.21,  web1 VM, 256 M内存。   
192.168.33.22,  web2 VM, 256 M内存。   


#### 首次部署(推送)
作用范畴如下:    

![/images/46-ansible-playbook-haproxy-nginx.gif](/images/46-ansible-playbook-haproxy-nginx.gif)    

环境部署后，load balance引导的网站访问流如下:    

![/images/46-vagrant-forwarded-port.gif](/images/46-vagrant-forwarded-port.gif)   

此工作流基于传统的推送，因而在这里不做太多的阐述。    

#### 配置更新(拉取) 
![/images/2017_06_30_09_58_43_1050x503.jpg](/images/2017_06_30_09_58_43_1050x503.jpg)

在配置好的lb, web1, web2机器上启用ansible-pull模式的指令如下:    

```
$ ansible-pull -i /var/lib/ansible/local/inventory.ini -d /var/lib/ansible/local -U http://root@192.168.33.2/root/centraladmin_lb.git
```
我们可以将它安装到crontab任务中，每次定时执行：    

```
*/3 * * * * root ansible-pull -i /var/lib/ansible/local/inventory.ini -d /var/lib/ansible/local -U http://root@192.168.33.2/root/centraladmin_lb.git -o >>/var/log/ansible-pull.log 2>&1
```

代码库维护, 举`centraladmin_web`为例,
创建该仓库时，我们需要制定其类型为`Public`才可，否则ansible-pull无法直接取回代码:    

![/images/2017_06_30_10_07_34_645x619.jpg](/images/2017_06_30_10_07_34_645x619.jpg)

代码中，可以指定需要运行的`local.yml`, 在其中添加需要定制化的配置。   

### 示例
