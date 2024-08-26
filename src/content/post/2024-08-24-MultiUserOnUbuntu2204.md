+++
title= "MultiUserOnUbuntu2204"
date = "2024-08-24T08:15:54+08:00"
description = "MultiUserOnUbuntu2204"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
Change hostname and print system info:    

```
# hostnamectl set-hostname multiubuntu
# exit
logout
Connection to 192.168.1.52 closed.
$ ssh root@192.168.1.52
root@multiubuntu:~# uname -a
Linux multiubuntu 5.15.0-117-generic #127-Ubuntu SMP Fri Jul 5 20:13:28 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
root@multiubuntu:~# cat /etc/issue
Ubuntu 22.04.4 LTS \n \l
```
Install xfce4(Using xfce4 for demo):    

```
# apt install -y xfce4
```
Create 10 test users, change shell, change passwd:     

```
$ for i in {1..10}; do useradd -m test$i; done
$ sudo ls /home
test  test1  test10  test2  test3  test4  test5  test6  test7  test8  test9
$ for i in {1..10}; do chsh -s /bin/bash test$i; done
$ vim bulkpasswords
$ chpasswd < bulkpasswords 
```
Define every user's `.xinitrc`:     

```
# cp /etc/X11/xinit/xinitrc .
# vim xinitrc
    ......
    # invoke global X session script
    #. /etc/X11/Xsession
    exec dbus-run-session -- startxfce4
# for i in {1..10}; do  cp xinitrc /home/test$i/.xinitrc; done
```
Change the available ttys:    

```
# vim /etc/systemd/logind.conf
[Login]
NAutoVTs=50
ReserveVT=50
```
Change the bash profile for each user:    

```
# cat bash_profile_example
#
# ~/.bash_profile
#

[[ -f ~/.bashrc ]] && . ~/.bashrc
if [ -z "$DISPLAY" ] && [ "$XDG_VTNR" = TOBEREPLACED ]; then
  exec startx &>/dev/null
fi
# for i in {1..10}; do cp /root/bash_profile_example /home/test$i/.bash_profile; sed -i "s/TOBEREPLACED/$i/g" /home/test$i/.bash_profile; chown -R test$i /home/test$i/.bash_profile; done
```
Add all user to autologin:     

```
# groupadd -r autologin
# for i in {1..10}; do gpasswd -a test$i autologin; done
```

autologin template file:   

```
[Service]
Type=simple
ExecStart=
ExecStart=-/usr/sbin/agetty --skip-login --nonewline --noissue --autologin TOBEREPLACED --noclear %I $TERM
```

Add autologin service:      

```

root@multi:~# cat autologin.conf 
[Service]
Type=simple
ExecStart=
ExecStart=-/usr/bin/agetty --skip-login --nonewline --noissue --autologin TOBEREPLACED --noclear %I $TERM
# for i in {1..50}; do mkdir -p /etc/systemd/system/getty\@tty$i.service.d/; done
for i in {1..50}; do cp autologin.conf /etc/systemd/system/getty\@tty$i.service.d/; sed -i "s/TOBEREPLACED/test$i/g" /etc/systemd/system/getty\@tty$i.service.d/autologin.conf; done
```
Now add autologin crontab:    

```
# crontab -e
@reboot sleep 10 && /usr/bin/chvtsh.sh
```
Create `chvtsh.sh`:     

```
#!/bin/bash
while ! ps -p $(pgrep Xorg) > /dev/null; do
  echo "Waiting for xorg to start...">>/tmp/waitdone.txt
  sleep 1
done
echo "xorg started, switching to console 2">>/tmp/waitdone.txt
sleep 5
for i in {2..50}
do
	chvt $i
	systemctl start getty@tty$i
	sleep 5
done
```
Enable the getty@tty1:     

```
# systemctl set-default multi-user.target
# systemctl enable getty@tty1
```
