+++
title= "WorkingTipsInRonggraphInLXD"
date = "2020-10-22T11:40:51+08:00"
description = "WorkingTipsInRonggraphInLXD"
keywords = ["Technology"]
categories = ["Technology"]
+++
### lxd environment
Install lxd(Offline):    

```
snap download core
snap download core18
snap download lxd
snap ack core18_1885.assert; snap ack core_10185.assert; snap ack lxd_17936.assert
snap install core18_1885.snap ; snap install core_10185.snap ; snap install lxd_17936.snap
dpkg -i ./lxd_1%3a0.9_all.deb
which lxc
which lxd
```
Show lxc images:    

```
root@rong320-1:~/lxd# lxc image list
If this is your first time running LXD on this machine, you should also run: lxd init
To start your first instance, try: lxc launch ubuntu:18.04

+-------+-------------+--------+-------------+--------------+------+------+-------------+
| ALIAS | FINGERPRINT | PUBLIC | DESCRIPTION | ARCHITECTURE | TYPE | SIZE | UPLOAD DATE |
+-------+-------------+--------+-------------+--------------+------+------+-------------+
```
Download lxd images:    

```
https://us.images.linuxcontainers.org/images/alpine/3.12/amd64/default/20201021_13:00/
Download
rootfs.squashfs lxd.tar.xz 
root@rong320-1:~/lxdimages# ls
lxd.tar.xz  rootfs.squashfs
root@rong320-1:~/lxdimages# lxc image import lxd.tar.xz rootfs.squashfs --alias alpine312
Image imported with fingerprint: 76560d125792d7710d70f41b060e81f0bd4d83f1cc4e8dbd43fc371e5dea27bf
root@rong320-1:~/lxdimages# lxc image list
+-----------+--------------+--------+------------------------------------------+--------------+-----------+--------+------------------------------+
|   ALIAS   | FINGERPRINT  | PUBLIC |               DESCRIPTION                | ARCHITECTURE |   TYPE    |  SIZE  |         UPLOAD DATE          |
+-----------+--------------+--------+------------------------------------------+--------------+-----------+--------+------------------------------+
| alpine312 | 76560d125792 | no     | Alpinelinux 3.12 x86_64 (20201021_13:00) | x86_64       | CONTAINER | 2.40MB | Oct 22, 2020 at 3:48am (UTC) |
+-----------+--------------+--------+------------------------------------------+--------------+-----------+--------+------------------------------+
```
Auto Config the lxd(https://discuss.linuxcontainers.org/t/usage-of-lxd-init-preseed/1069/3)
(https://lxd.readthedocs.io/en/latest/preseed/)
:    

```
cat <<EOF | lxd init --preseed
config:
  core.https_address: 10.137.149.161:9199
  images.auto_update_interval: 15
networks:
- name: lxdbr0
  type: bridge
  config:
    ipv4.address: auto
    ipv6.address: none
EOF
root@rong320-1:~/lxdimages# cat storages.yml 
storage_pools:
- name: default
  driver: dir
  config:
    source: ""
root@rong320-1:~/lxdimages# lxd init --preseed<./storages.yml

root@rong320-1:~/lxdimages# cat profiles.yml 
profiles:
- name: default
  devices:
    root:
      path: /
      pool: default
      type: disk
    eth0:
      nictype: bridged
      parent: lxdbr0
      type: nic
root@rong320-1:~/lxdimages# lxd init --preseed<profiles.yml

```
Now we could check the default lxd bridges(lxdbr0).  
### Docker/Docker-compose in alpine
Create a new profile named `k8s`:    

```
# lxc launch alpine312 firstalpine -p k8s
```

Create the first alpine instance:   

```
# lxc launch alpine312 firstalpine
Creating firstalpine
Starting firstalpine           
# lxc ls
+-------------+---------+---------------------+------+-----------+-----------+
|    NAME     |  STATE  |        IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------+---------+---------------------+------+-----------+-----------+
| firstalpine | RUNNING | 10.31.47.210 (eth0) |      | CONTAINER | 0         |
+-------------+---------+---------------------+------+-----------+-----------+
root@rong320-1:~/lxdimages# lxc exec firstalpine /bin/sh
~ # cat /etc/issue
Welcome to Alpine Linux 3.12
Kernel \r on an \m (\l)
```
Configure repository:   

```
 # echo "https://mirrors.aliyun.com/alpine/v3.12/main/" > /etc/apk/repositories
 # echo "https://mirrors.aliyun.com/alpine/v3.12/community/" >> /etc/apk/repositories
# apk update
# apk add docker-engine docker-compose docker-cli
```
Create the `cgroups-patch` file under `/etc/init.d`:    

```
#!/sbin/openrc-run

description="Mount the control groups for Docker"

depend()
{
    keyword -docker
    need sysfs cgroups
}

start()
{
    if [ -d /sys/fs/cgroup ]; then
        mkdir -p /sys/fs/cgroup/cpu,cpuacct
        mkdir -p /sys/fs/cgroup/net_cls,net_prio

        mount -n -t cgroup cgroup /sys/fs/cgroup/cpu,cpuacct -o rw,nosuid,nodev,noexec,relatime,cpu,cpuacct
        mount -n -t cgroup cgroup /sys/fs/cgroup/net_cls,net_prio -o rw,nosuid,nodev,noexec,relatime,net_cls,net_prio

        if ! mountinfo -q /sys/fs/cgroup/openrc; then
            local agent="${RC_LIBEXECDIR}/sh/cgroup-release-agent.sh"
            mkdir -p /sys/fs/cgroup/openrc
            mount -n -t cgroup -o none,nodev,noexec,nosuid,name=systemd,release_agent="$agent" openrc /sys/fs/cgroup/openrc
        fi
    fi

    return 0
}

```
Added the auto-start and reboot:    

```
# rc-update add cgroups-patch boot
# vim /etc/init.d/docker
.....
start_pre() {
        #checkpath -f -m 0644 -o root:docker "$DOCKER_ERRFILE" "$DOCKER_OUTFILE"
        echo "fucku"
}
.....
# rc-service docker start
# rc-update add docker default
# reboot
After reboot, check docker version
```

push files into lxc instance:    

```
# lxc file push -r podmanitems/ firstalpine/root/
load all images
# docker images
~ # docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
rong/ui             master              66ad16eb15c5        20 minutes ago      28.9MB
rong/server         master              8150777ead18        23 hours ago        301MB
rong/kobe           master              2d0a03d6cedb        2 days ago          231MB
rong/nginx          1.19.2-amd64        7e4d58f0e5f3        5 weeks ago         133MB
rong/webkubectl     v2.6.0-amd64        4aa634837fea        2 months ago        349MB
rong/mysql-server   8.0.21-amd64        8a3a24ad33be        3 months ago        366MB
# lxc file push ronggraph.tar firstalpine/root/
# tar xzvf /root/ronggraph.tar
```
Should write a start definition for ronggraph:    

```
#!/sbin/openrc-run
#
# author: Yusuke Kawatsu

workspace="/root/ronggraph"
cmdpath="/usr/bin/docker-compose"
prog="ronggraph"
lockfile="/var/lock/ronggraph"
pidfile="/var/run/ronggraph.pid"
PATH="$PATH:/usr/local/bin"


start() {
    [ -x $cmdpath ] || exit 5
    echo -n $"Starting $prog: "

    cd $workspace
    $cmdpath up -d
    $cmdpath down
    retval=$?
    pid=$!
    echo
    [ $retval -eq 0 ] && touch $lockfile && echo $pid > $pidfile

    return $retval
}

stop() {
    [ -x $cmdpath ] || exit 5
    echo -n $"Stopping $prog: "

    cd $workspace
    $cmdpath down
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile && rm -f $pidfile

    return $retval
}

restart() {
    stop
    sleep 3
    start
}

depend() {
    need docker
}

```
Now  add ronggraph to default update:   

```
# rc-update add ronggraph default
# halt
```

Save the current status:    

```
root@rong320-1:~/lxdimages# lxc stop firstalpine
root@rong320-1:~/lxdimages# lxc publish --public firstalpine --alias=ronggraph
root@rong320-1:/mnt# lxc image ls
+-----------+--------------+--------+------------------------------------------+--------------+-----------+----------+------------------------------+
|   ALIAS   | FINGERPRINT  | PUBLIC |               DESCRIPTION                | ARCHITECTURE |   TYPE    |   SIZE   |         UPLOAD DATE          |
+-----------+--------------+--------+------------------------------------------+--------------+-----------+----------+------------------------------+
| alpine312 | 76560d125792 | no     | Alpinelinux 3.12 x86_64 (20201021_13:00) | x86_64       
| CONTAINER | 2.40MB   | Oct 22, 2020 at 6:18am (UTC) |
+-----------+--------------+--------+------------------------------------------+--------------+-----------+----------+------------------------------+
| ronggraph | b31788790460 | yes    | Alpinelinux 3.12 x86_64 (20201021_13:00) | x86_64       | CONTAINER | 619.40MB | Oct 22, 2020 at 8:05am (UTC) |
+-----------+--------------+--------+------------------------------------------+--------------+-----------+----------+------------------------------+
```
launch new instance:    

```
# lxc launch ronggraph ronggraph -p k8s
# 
```
Add forward rules:    

```
lxc config device add ronggraph myport80 proxy listen=tcp:0.0.0.0:80 connect=tcp:0.0.0.0:80
lxc config device add ronggraph myport443 proxy listen=tcp:0.0.0.0:443 connect=tcp:0.0.0.0:443
```

### arm64 workingtips
Under rpi archlinux64, install:     

```
# pacman -Sy
# pacman -S lxc lxd
# systemctl enable lxd
# systemctl start lxd
```
Download images from:     

```
https://us.images.linuxcontainers.org/images/alpine/3.12/arm64/default/20201022_13:00/
rootfs.squashfs
lxd.tar.xz
# lxc image import lxd.tar.xz rootfs.squashfs --alias alpine312
# lxd init --preseed<pre-rong/lxditems/lxd_snap/init.yaml
# lxc profile create k8s
# lxc profile edit k8s<pre-rong/lxditems/lxdimages/k8s.yaml
```
![/images/2020_10_23_09_55_43_629x378.jpg](/images/2020_10_23_09_55_43_629x378.jpg)

Configure lxc for running:    

```
https://wiki.archlinux.org/index.php/Linux_Containers
```
lxc Installation:   

```
# ~ # sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
~ # cat /etc/apk/repositories 
http://mirrors.ustc.edu.cn/alpine/v3.12/main
http://mirrors.ustc.edu.cn/alpine/v3.12/community
Install docker/docker-compose, modify its startup
```

lxc public will take a very long time!!!
