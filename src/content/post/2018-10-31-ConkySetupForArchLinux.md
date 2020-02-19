+++
title = "ConkySetupForArchLinux"
date = "2018-10-31T15:57:32+08:00"
description = "ConkySetupForArchLinux"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Font Preparation
Download `Conky Icons by Carelli.ttf` from following url:    

```
https://github.com/antoniocarelli/conky/blob/master/Conky%20Icons%20by%20Carelli.ttf
```

With the downloaded ttf, do following command:    

```
$ cp fonts/Conky\ Icons\ by\ Carelli.ttf /usr/share/fonts/TTF/
$ cd /usr/share/fonts/TTF/
$ sudo chmod 0444 Conky\ Icons\ by\ Carelli.ttf
$ sudo fc-cache
$ fc-match -a | grep -i conky
Conky Icons by Carelli.ttf: "Conky Icons by Carelli" "Regular"
```
Install font-awesome:    

```
$ yaourt -S ttf-font-awesome-4 --noconfirm
```
Most conky configuration files use font-awesome version 4  rather than version
5, so we should install specified version 4.   
### Configuration File
My conky configuration file is listed under :    

```
https://gist.githubusercontent.com/purplepalmdash/4078219891aa60923e16f4a98bd9bffb/raw/d6a5711a839dace3ce440ad6bc061691a32941bf/conky.conf
```

So simply download this file and run conky via:    

```
$ cd ~ && wget github_gist/conky.conf
$ conky -c ~/conky.conf
```
Now you will see the conky running like:    

![/images/2018_10_31_16_05_46_438x911.jpg](/images/2018_10_31_16_05_46_438x911.jpg)

Add conky into startup file:    

```
$ vim ~/.config/awesome/rc.lu
run_once("conky -c /home/xxxxx/.conky/conky.conf.3 &")
```

### Transparent
In case you want to enable transparent for awesome wm, change to following
configuration:    

```
	text_buffer_size = 32768,
	imlib_cache_size = 0,
	own_window = true,
	own_window_type = 'override',
        own_window_class = "Conky",
	own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',
	own_window_transparent = true,
	border_inner_margin = 10,
	border_outer_margin = 0,
	xinerama_head = 2,
	alignment = 'top_left',
	gap_x = 1185,
	gap_y = 35,
	draw_shades = false,
	draw_outline = false,
	draw_borders = false,
	draw_graph_borders = false,
```
Then the result will be displayed like following:    

![/images/2018_10_31_16_44_03_613x870.jpg](/images/2018_10_31_16_44_03_613x870.jpg)

The full configuration for transparent is listed as:    

```
https://gist.github.com/purplepalmdash/753caaa124d9da0d58b3a7b08738d8fa
```
