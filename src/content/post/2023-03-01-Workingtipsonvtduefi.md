+++
title= "Workingtipsonvtduefi"
date = "2023-03-01T10:53:12+08:00"
description = "Workingtipsonvtduefi"
keywords = ["Technology"]
categories = ["Technology"]
+++
Change the default ovmf files:    

```
cd /usr/share/OVMF
mv OVMF_CODE.fd OVMF_CODE.fd.official
cp /home/idv/10th/0623.fd OVMF_CODE.fd
```
To be continued.
