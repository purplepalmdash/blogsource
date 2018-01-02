+++
categories = ["Linux"]
date = "2016-08-03T09:55:58+08:00"
description = "Directly send message to Slack"
keywords = ["Linux"]
title = "SendMessageToSlack"

+++
### Reference
Mainly refers to:    

[https://api.slack.com/incoming-webhooks](https://api.slack.com/incoming-webhooks)     

[http://blog.pragbits.com/it/2015/02/09/slack-notifications-via-curl/](http://blog.pragbits.com/it/2015/02/09/slack-notifications-via-curl/)    

### Incomming webhooks
Create a incoming webhooks in:    
[https://skyruntime.slack.com/apps/new/A0F7XDUAZ-incoming-webhooks](https://skyruntime.slack.com/apps/new/A0F7XDUAZ-incoming-webhooks)   
 
![/images/2016_08_03_09_57_54_920x296.jpg](/images/2016_08_03_09_57_54_920x296.jpg)    

After created the webhook use curl for sending out the direct message:    

```
$ curl -X POST --data-urlencode 'payload={"channel": "#general", "username":
"webhookbot", "text": "This is posted to #general and comes from a bot named
webhookbot.", "icon_emoji": ":ghost:"}'
https://hooks.slack.com/services/xxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Wrap the message
In Bash:    

```
$ proxychains4 curl -X POST --data-urlencode 'payload={"channel": "#general", "username":
"webhookbot", "text": "'$1'", "icon_emoji": ":ghost:"}'
https://hooks.slack.com/services/xxxxxxxxxxxxxxxxxxxxxxxxx
```
Turn it into a executable file under `bin`, then you could use command like this:    

```
$ command1; sendSlack 'fcuk...'
```
