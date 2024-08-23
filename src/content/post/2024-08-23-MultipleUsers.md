+++
title= "MultipleUsers"
date = "2024-08-23T15:43:27+08:00"
description = "MultipleUsers"
keywords = ["Technology"]
categories = ["Technology"]
+++
Create users:     

```
for i in {1..20}; do  useradd -m test$i; done
```
Create a bulkpasswords file:     

```
test1: goeugoogue
test2: gowguowgow
.....
```
Change password:     

```
chpasswd < bulkpasswords 
```
Copy the getty service:    

```
 for i in {7..21}; do cp -r getty\@tty6.service.d/ getty\@tty$i.service.d; done
```
change the systemd files:    

```
[root@archremote system]# vim getty\@tty7.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty6.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty8.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty9.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty10.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty11.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty12.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty13.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty14.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty15.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty16.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty17.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty18.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty19.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty20.service.d/autologin.conf 
[root@archremote system]# vim getty\@tty6.service.d/autologin.conf 
[root@archremote system]# cat getty\@tty6.service.d/autologin.conf 
[Service]
Type=simple
ExecStart=
#ExecStart=-/sbin/agetty -o '-p -f -- \\u' --noclear --autologin dash %I $TERM
ExecStart=-/usr/bin/agetty --skip-login --nonewline --noissue --autologin test6 --noclear %I $TERM
```


```
for i in {6..20}; do cp /home/test1/.bash_profile /home/test$i/.bash_profile && chown -R test$i /home/test$i/.bash_profile; done
vim /home/test6/.bash_profile 
vim /home/test7/.bash_profile 
vim /home/test8/.bash_profile 
vim /home/test9/.bash_profile 
vim /home/test10/.bash_profile 
vim /home/test11/.bash_profile 
vim /home/test12/.bash_profile 
vim /home/test13/.bash_profile 
vim /home/test14/.bash_profile 
vim /home/test15/.bash_profile 
vim /home/test16/.bash_profile 
vim /home/test17/.bash_profile 
vim /home/test18/.bash_profile 
vim /home/test19/.bash_profile 
vim /home/test20/.bash_profile 
for i in {6..20}; do cp /home/test1/.xinitrc /home/test$i/.xinitrc; chmod 777 /home/test$i/.xinitrc; done
```
Example for `bash_profile`:    

```
[root@archremote ~]# cat /home/test18/.bash_profile
#
# ~/.bash_profile
#

[[ -f ~/.bashrc ]] && . ~/.bashrc
if [ -z "$DISPLAY" ] && [ "$XDG_VTNR" = 18 ]; then
  exec startx &>/dev/null
fi
```
Added autologin:    

```
 gpasswd -a test7 autologin
 for i in {8..20}; do gpasswd -a test$i autologin; done
```

crontab for activating this:     

```
[root@archremote ~]# cat /usr/bin/chvtsh.sh 
#!/bin/bash
while ! ps -p $(pgrep Xorg) > /dev/null; do
  echo "Waiting for xorg to start...">>/tmp/waitdone.txt
  sleep 1
done
echo "xorg started, switching to console 2">>/tmp/waitdone.txt
sleep 15
chvt 2
sleep 5
chvt 3
sleep 5
chvt 4
sleep 5
chvt 5
sleep 5
chvt 6
sleep 5
chvt 7
systemctl start getty@tty7
sleep 5
chvt 8
systemctl start getty@tty8
sleep 5
chvt 9
systemctl start getty@tty9
[root@archremote ~]# crontab -l
@reboot sleep 10 && /usr/bin/chvtsh.sh
```
