+++
title = "OnFIOTesting"
date = "2017-08-21T09:22:57+08:00"
description = "On fio testing"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Environment
bare metal vs kvm vs docker
Testing script:    

```
#!/bin/bash
function tgt_r {
fio -filename=/root/ioscript/ccc -direct=1 -iodepth 4 -thread -rw=read -ioengine=libaio -bs=$1 -size=120G  -runtime=200 -group_reporting -name=mytest &>> s_r_test
}

function tgt_w {
fio -filename=/root/ioscript/ccc -direct=1 -iodepth 4 -thread -rw=write -ioengine=libaio -bs=$1 -size=120G  -runtime=200 -group_reporting -name=mytest &>> s_w_test
}


function tgt_rr {
fio -filename=/root/ioscript/ccc -direct=1 -iodepth 4 -thread -rw=randread -ioengine=libaio -bs=$1 -size=120G  -runtime=200 -group_reporting -name=mytest &>> r_r_test
}

function tgt_rw {
fio -filename=/root/ioscript/ccc -direct=1 -iodepth 4 -thread -rw=randwrite -ioengine=libaio -bs=$1 -size=120G  -runtime=200 -group_reporting -name=mytest &>> r_w_test
}

mkdir -p /root/ioscript
rm -f /root/ioscript/ccc; touch /root/ioscript/ccc
tgt_r 128K
rm -f /root/ioscript/ccc; touch /root/ioscript/ccc
tgt_w 128K
rm -f /root/ioscript/ccc; touch /root/ioscript/ccc
tgt_rr 4K
rm -f /root/ioscript/ccc; touch /root/ioscript/ccc
tgt_rw 4K
```
Sequence read/write, 128K. Random read/write: 4K.   

KVM using block device:   

![/images/2017_08_21_09_24_41_484x416.jpg](/images/2017_08_21_09_24_41_484x416.jpg)

###  Result:    
Bare Metal:    

```
Sequence Read: 1000
Sequence Write: 893
Random Read: 1000
Random Write: 131 
```

KVM:    

```
Sequence Read: 800
Sequence Write: 795
Random Read: 84
Random Write: 107
```

Docker:   

```
Sequence Read: 975
Sequence Write: 909
Random Read: 114
Random Write: 81
```
Seems Not OK.......

Will use another disk for testing.    
