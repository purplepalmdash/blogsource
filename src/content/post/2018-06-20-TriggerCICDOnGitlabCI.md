+++
title = "TriggerCICDOnGitlabCI"
date = "2018-06-20T11:30:25+08:00"
description = "TriggerCICDOnGitlabCI"
keywords = ["Linux"]
categories = ["Technology"]
+++
Projects list:    

![/images/2018_06_20_11_34_53_645x361.jpg](/images/2018_06_20_11_34_53_645x361.jpg)

We will let kismatic_source trigger buildiso.    

The `.gitlab-ci.yml` listed is:   

![/images/2018_06_20_11_36_39_886x329.jpg](/images/2018_06_20_11_36_39_886x329.jpg)

The build-iso's `.gitlab-ci.yml` is listed as:    

![/images/2018_06_20_11_37_18_886x137.jpg](/images/2018_06_20_11_37_18_886x137.jpg)

Settings->CI/CD->Pipeline triggers, 

![/images/2018_06_20_11_38_21_867x528.jpg](/images/2018_06_20_11_38_21_867x528.jpg)

Then you could trigger the building iso via push sources to kismatic_images.    
