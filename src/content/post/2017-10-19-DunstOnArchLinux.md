+++
title = "DunstOnArchLinux"
date = "2017-10-19T11:14:59+08:00"
description = "dunst on archlinux"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Installation
My desktop environment is `awesome`, my operating system is ArchLinux.    

dunst and dunst-git are both ok for installation, because dunst-git have the
systemd file, which would be very convenient for configuration, I install it
via:    

```
$ yaourt dunst-git
```
### Configuration
Enable the user's systemd configuration via:    

```
$ systemctl --user enable dunst
```

But your dunst will start-up with errors, calling:    

`"Name Lost. Is Another notification daemon running?"`

This is because awesome has its own desktop notification daemon called
`naughty`, simply comment them out in your `~/.config/awesome/rc.lua`, my
example is listed as in following diff files:    

```
10c10
< --* local naughty = require("naughty")
---
> local naughty = require("naughty")
14,37c14,37
< --* -- {{{ Error handling
< --* -- Check if awesome encountered an error during startup and fell back to
< --* -- another config (This code will only ever execute for the fallback config)
< --* if awesome.startup_errors then
< --*     naughty.notify({ preset = naughty.config.presets.critical,
< --*                      title = "Oops, there were errors during startup!",
< --*                      text = awesome.startup_errors })
< --* end
< --* 
< --* -- Handle runtime errors after startup
< --* do
< --*     local in_error = false
< --*     awesome.connect_signal("debug::error", function (err)
< --*         -- Make sure we don't go into an endless error loop
< --*         if in_error then return end
< --*         in_error = true
< --* 
< --*         naughty.notify({ preset = naughty.config.presets.critical,
< --*                          title = "Oops, an error happened!",
< --*                          text = tostring(err) })
< --*         in_error = false
< --*     end)
< --* end
< --* -- }}}
---
> -- {{{ Error handling
> -- Check if awesome encountered an error during startup and fell back to
> -- another config (This code will only ever execute for the fallback config)
> if awesome.startup_errors then
>     naughty.notify({ preset = naughty.config.presets.critical,
>                      title = "Oops, there were errors during startup!",
>                      text = awesome.startup_errors })
> end
> 
> -- Handle runtime errors after startup
> do
>     local in_error = false
>     awesome.connect_signal("debug::error", function (err)
>         -- Make sure we don't go into an endless error loop
>         if in_error then return end
>         in_error = true
> 
>         naughty.notify({ preset = naughty.config.presets.critical,
>                          title = "Oops, an error happened!",
>                          text = tostring(err) })
>         in_error = false
>     end)
> end
> -- }}}
```
Now restart your awesome, you will get the dunst working, test it via
`notify-send "Title" "Now"`, you will see:    

![/images/2017_10_19_11_21_16_370x108.jpg](/images/2017_10_19_11_21_16_370x108.jpg)


### Crontab
Using crontab for sending notifications, first we create the scripts for
sending the notifications:    


```
➜  ~ cat /bin/notify.sh 
############# crontab ###################
#### every 1 hour between 9:00 am - 18:00 pm
#0 9-18/1 * * * /path/command
#### at certain time #####
#10 9-18/1 * * * /path/command
############# Standup boy ################
current_time=`date`

############# To-Do List #################
filename="/home/dash/tasks.txt"
filecontent=`cat $filename`

#### Until you click it, you won't get this window vanish #####
notify-send -u critical -t 0 "$current_time, Stand UP, Boy" "$filecontent"

#### example for read every lines #### 
#while read -r line
#do
#    #notify-send "$line"
#done < "$filename"
```

Then we install and configure crontab for hourly run this script:    

```
$ sudo pacman -S cronie
$ sudo systmctl enable cronie.service
$ crontab -e
$ sudo systemctl start cronie.service
```

The crontab items:    

```
$ crontab -l 
10 9-18/1 * * * /bin/notify.sh
```
