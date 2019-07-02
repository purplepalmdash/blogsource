+++
title = "Arm64ISOCustomization"
date = "2019-07-02T09:51:48+08:00"
description = "Arm64ISOCustomization"
keywords = ["Linux"]
categories = ["Linux"]
+++
Make working directory:     

```
#  mkdir Rong1907iso
# cd Rong1907iso/
# cp ../ubuntu-18.04.2-server-arm64.iso .
# cp -r ./iso/* ./newISO
# cp -r ./iso/.disk ./newISO
# umount ./iso
# rm -f ubuntu-18.04.2-server-arm64.iso 
# rm -rf iso/
```

