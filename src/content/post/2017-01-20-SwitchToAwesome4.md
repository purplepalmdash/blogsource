+++
date = "2017-01-20T14:07:37+08:00"
description = "Switch to awesome 4.0"
title = "Awesome4.0 配置"
categories = ["Linux"]
keywords = ["awesome"]

+++
### Aim
For Awesome is upgrading to 4.0 version, the configuration file has changed
a lot comparing to Awesome 3.5, I have to rewrite the configuration files,
following records all of the steps for customize Awesome 4.0.     

### Configure File
Simply copy the configuration file from xdg:    

```
$ cp /etc/xdg/awesome/rc.lua ~/.config/awesome/
```
### Customization
#### default terminal
Change default terminal from xterm to gnome-terminal:    

```
-- terminal = "xterm"
terminal = "gnome-terminal"
```

#### Run Once Programs
Add one function then call it:    

```
--- Just Run Once Programs.
function run_once(cmd)
  findme = cmd
  firstspace = cmd:find(" ")
  if firstspace then
    findme = cmd:sub(0, firstspace-1)
  end
  awful.util.spawn_with_shell("pgrep -u $USER -x " .. findme .. " > /dev/null || (" .. cmd .. ")")
end

run_once("udiskie -2 --tray")
run_once("wicd-client --tray")
run_once("fcitx &")
run_once("variety")
run_once("whatpulse")
```

#### Change tag name
Change the default `1 2 3 4 5` to you own tag name list:    

```
    -- Each screen has its own tag table.
    -- awful.tag({ "1", "2", "3", "4", "5", "6", "7", "8", "9" }, s, awful.layout.layouts[1])
    local names = { "1WWW", "2Code", "3Term", "4Office", "5", "6", "7", "8", "9IM" }
    local l = awful.layout.suit  -- Just to save some typing: use an alias.
    local layouts = { l.floating, l.tile, l.floating, l.fair, l.max, l.floating, l.tile.left, l.floating, l.floating }
    awful.tag(names, s, layouts)
```

#### Use d-menu
When press `mod4 + r`, change it to calling d-menu:    

```
    -- Prompt
    -- awful.key({ modkey },            "r",     function () awful.screen.focused().mypromptbox:run() end,
    --           {description = "run prompt", group = "launcher"}),

    awful.key({ modkey },            "r",     function ()
    awful.util.spawn("dmenu_run -i -p 'Run command:' -nb '" .. 
 		beautiful.bg_normal .. "' -nf '" .. beautiful.fg_normal .. 
		"' -sb '" .. beautiful.bg_focus .. 
		"' -sf '" .. beautiful.fg_focus .. "'") 
	end),
```

