+++
title= "incusbaseddesktop"
date = "2024-10-15T10:15:30+08:00"
description = "incusbaseddesktop"
keywords = ["Technology"]
categories = ["Technology"]
+++
### tips
Adjust the memory for incus instance:    

```
incus launch images:ubuntu/22.04 first
incus config set first limits.memory=8GiB
# free -m shows 8192
incus config unset first limits.memory
# free -m shows the host memory size
```
view the config via:    

```
incus config show first
```
