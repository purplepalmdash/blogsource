+++
title= "EnablePulseAudioInSystemMode"
date = "2021-11-30T10:24:38+08:00"
description = "EnablePulseAudioInSystemMode"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目的
在Ubuntu `18.04.6`上配置pulseaudio的 `system-wide` daemon模式，以使得`pulseaudio`为所有用户可用。    

### 步骤
确保`pulseaudio`被安装（默认应该是被安装的), 撰写一个`systemd`服务条目，重新定义其启动方式(做完以下步骤后需要重新启动机器), 需注意需要手动执行`usermod`一行为所有用户添加到组里:    

```
# vi /etc/systemd/system/pulseaudio.service
[Unit]
Description=PulseAudio Daemon
 
[Install]
WantedBy=multi-user.target
 
[Service]
Type=simple
PrivateTmp=true
ExecStart=/usr/bin/pulseaudio --system --realtime --disallow-exit --no-cpu-limit 
# vi /usr/share/dbus-1/system.d/pulseaudio.conf 
<?xml version="1.0"?> <!--*-nxml-*-->
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
    <policy group="pulse">
        <allow own="org.pulseaudio.Server"/>
    </policy>

    <policy context="default">
        <allow send_destination="org.pulseaudio.Server"/>
        <allow receive_sender="org.pulseaudio.Server"/>
    </policy>
</busconfig>
# groupadd --system pulse
# groupadd --system pulse-access
# useradd --system -g pulse -G audio -d /var/run/pulse -m pulse
# usermod -G video,pulse-access root
# usermod -G video,pulse-access test
# usermod -G video,pulse-access seat1
# usermod -G video,pulse-access seat2
# echo "default-server = /var/run/pulse/native" >> /etc/pulse/client.conf
# echo "autospawn = no" >> /etc/pulse/client.conf
# systemctl daemon-reload
# systemctl enable pulseaudio
# reboot
```
重启后，以ssh登陆到各用户下，在命令行下播放音频，如用`mplayer 1.mp3`等操作，应该可以看到音频被正确解码，但是此时无声音，应该使用以下命令unmute音道.   

开启所有音道:   

```
# pactl set-sink-mute @DEFAULT_SINK@ false
```
