---
categories: ["Linux"]
comments: true
date: 2016-02-23T19:52:57Z
title: Using python for checking imap mailbox unread message
url: /2016/02/23/using-python-for-checking-imap-mailbox-unread-message/
---

Just paste some python scripts:    

```
>>> import imaplib
>>> obj=imaplib.IMAP4_SSL('imap.163.com','993')
>>> obj.login('XXXXX','XXXXXXXX')
('OK', [b'LOGIN completed'])
>>> obj.select()
('OK', [b'49'])
>>> obj.search(None,'Unseen')
('OK', [b''])
>>> len(obj.search(None, 'UnSeen')[1][0].split()) 
0
>>> len(obj.search(None, 'UnSeen')[1][0].split()) 
1
>>> len(obj.search(None, 'UnSeen')[1][0].split()) 
2
>>> len(obj.search(None, 'UnSeen')[1][0].split()) 
2
>>> len(obj.search(None, 'UnSeen')[1][0].split()) 
1
>>> 
```

Using the unread counts, you could easily interact with some other programs,
for example shining LEDs.    
