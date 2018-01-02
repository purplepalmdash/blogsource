+++
title = "CorrectHugoDate"
date = "2017-07-27T16:28:35+08:00"
description = "CorrectHugoDate"
keywords = ["Linux"]
categories = ["Linux"]
+++

### Problem
![/images/2017_07_27_16_08_05_1361x260.jpg](/images/2017_07_27_16_08_05_1361x260.jpg)

### Reason
This is because hugo upgrade to a new version `0.25.1`, while this new version
won't give the default value of date in newly created markdown file.    

### Solution
Edit the `themes/hyde-a/archetypes/default.md`, add following items:    

```
+++
title = ""
date = "{{ .Date }}"
description = ""
keywords = ["Linux"]
categories = ["Linux"]
+++
```

Now you could re-new your configuration, and then your blog will acts OK.   
