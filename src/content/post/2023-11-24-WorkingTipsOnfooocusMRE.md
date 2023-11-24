+++
title= "WorkingTipsOnfooocusMRE"
date = "2023-11-24T17:07:13+08:00"
description = "WorkingTipsOnfooocusMRE"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
Install steps:    

```
git clone https://github.com/MoonRide303/Fooocus-MRE.git
cd Fooocus-MRE
python3 -m venv fooocus_env
source fooocus_env/bin/activate
pip install pygit2==1.12.2
pip install --upgrade pip
pip3 install packaging
source fooocus_env/bin/activate
python entry_with_update.py --listen
```

