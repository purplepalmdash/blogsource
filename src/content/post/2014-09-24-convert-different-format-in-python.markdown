---
categories: ["null"]
comments: true
date: 2014-09-24T00:00:00Z
title: Convert different format in python
url: /2014/09/24/convert-different-format-in-python/
---

The following commands is for directly convert ascii, numbers in python:     

```
$ python2
>>> str(unichr(97))
'a'
>>> str(unichr(0x68))
'h'
>>> str(unichr(0x5a))
'Z'
>>> bin(0x2711)
'0b10011100010001'
>>> 0x2711
10001
>>> hex(1127)
'0x467'

```

