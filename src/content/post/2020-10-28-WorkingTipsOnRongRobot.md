+++
title= "WorkingTipsOnRongRobot"
date = "2020-10-28T08:27:00+08:00"
description = "WorkingTipsOnRongRobot"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Azure DevOps
Create a new project:     

![/images/2020_10_28_08_37_20_634x542.jpg](/images/2020_10_28_08_37_20_634x542.jpg)


Add the ssh-key into project:    

![/images/2020_10_28_08_31_38_457x200.jpg](/images/2020_10_28_08_31_38_457x200.jpg)

Configure the time/locale:   

![/images/2020_10_28_08_32_41_584x548.jpg](/images/2020_10_28_08_32_41_584x548.jpg)

### Repos
Create a new repository and set the remote branch:    

```
# mkdir RongRobot
# cd RongRobot
# vim README.md
# git init
# git add .
# git commit -m "First Commit"
# git remote add origin git@ssh.dev.azure.com:v3/purplepalm/RongRobot/RongRobot
# git push -u origin --all
```
View status on azure devops:   

![/images/2020_10_28_08_41_24_1023x408.jpg](/images/2020_10_28_08_41_24_1023x408.jpg)

Click `Set up build` for setup the pipeline:    

![/images/2020_10_28_08_42_18_317x141.jpg](/images/2020_10_28_08_42_18_317x141.jpg)

Starter pipeline:   

![/images/2020_10_28_08_42_48_518x262.jpg](/images/2020_10_28_08_42_48_518x262.jpg)

Edit something:    

![/images/2020_10_28_08_43_34_791x555.jpg](/images/2020_10_28_08_43_34_791x555.jpg)

### Codes
Write your own azure pipelines for doing these .
