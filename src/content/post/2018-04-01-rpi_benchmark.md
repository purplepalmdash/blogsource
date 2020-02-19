+++
title = "rpibenchmark"
date = "2018-04-01T21:40:07+08:00"
description = "rpibenchmark"
keywords = ["Linux"]
categories = ["Technology"]
+++
Install via:    

```
 $ git clone https://github.com/kdlucas/byte-unixbench.git
Cloning into 'byte-unixbench'...
remote: Counting objects: 204, done.
remote: Total 204 (delta 0), reused 0 (delta 0), pack-reused 204
Receiving objects: 100% (204/204), 198.85 KiB | 207.00 KiB/s, done.
Resolving deltas: 100% (105/105), done.
pi@raspberrypi:~ $ cd byte-unixbench/UnixBench/

```

Run steps:   

```
pi@raspberrypi:~/byte-unixbench/UnixBench $ ./Run 
gcc -o pgms/arithoh -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -Darithoh src/arith.c 
gcc -o pgms/register -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -Ddatum='register int' src/arith.c 
gcc -o pgms/short -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -Ddatum=short src/arith.c 
gcc -o pgms/int -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -Ddatum=int src/arith.c 
gcc -o pgms/long -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -Ddatum=long src/arith.c 
gcc -o pgms/float -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -Ddatum=float src/arith.c 
gcc -o pgms/double -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -Ddatum=double src/arith.c 
gcc -o pgms/hanoi -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/hanoi.c 
gcc -o pgms/syscall -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/syscall.c 
gcc -o pgms/context1 -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/context1.c 
gcc -o pgms/pipe -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/pipe.c 
gcc -o pgms/spawn -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/spawn.c 
gcc -o pgms/execl -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/execl.c 
gcc -o pgms/dhry2 -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -DHZ= ./src/dhry_1.c ./src/dhry_2.c
gcc -o pgms/dhry2reg -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -DHZ= -DREG=register ./src/dhry_1.c ./src/dhry_2.c
gcc -o pgms/looper -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/looper.c 
gcc -o pgms/fstime -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME src/fstime.c 
gcc -o pgms/whetstone-double -Wall -pedantic -O3 -ffast-math -march=native -mtune=native -I ./src -DTIME -DDP -DGTODay -DUNIXBENCH src/whets.c -lm
make all
make[1]: Entering directory '/home/pi/byte-unixbench/UnixBench'
make distr
make[2]: Entering directory '/home/pi/byte-unixbench/UnixBench'
Checking distribution of files
./pgms  exists
./src  exists
./testdir  exists
./tmp  exists
./results  exists
make[2]: Leaving directory '/home/pi/byte-unixbench/UnixBench'
make programs
make[2]: Entering directory '/home/pi/byte-unixbench/UnixBench'
make[2]: Nothing to be done for 'programs'.
make[2]: Leaving directory '/home/pi/byte-unixbench/UnixBench'
make[1]: Leaving directory '/home/pi/byte-unixbench/UnixBench'
sh: 1: 3dinfo: not found

   #    #  #    #  #  #    #          #####   ######  #    #   ####   #    #
   #    #  ##   #  #   #  #           #    #  #       ##   #  #    #  #    #
   #    #  # #  #  #    ##            #####   #####   # #  #  #       ######
   #    #  #  # #  #    ##            #    #  #       #  # #  #       #    #
   #    #  #   ##  #   #  #           #    #  #       #   ##  #    #  #    #
    ####   #    #  #  #    #          #####   ######  #    #   ####   #    #

   Version 5.1.3                      Based on the Byte Magazine Unix Benchmark

   Multi-CPU version                  Version 5 revisions by Ian Smith,
                                      Sunnyvale, CA, USA
   January 13, 2011                   johantheghost at yahoo period com

------------------------------------------------------------------------------
   Use directories for:
      * File I/O tests (named fs***) = /home/pi/byte-unixbench/UnixBench/tmp
      * Results                      = /home/pi/byte-unixbench/UnixBench/results
------------------------------------------------------------------------------

Use of uninitialized value in printf at ./Run line 1469.
Use of uninitialized value in printf at ./Run line 1470.
Use of uninitialized value in printf at ./Run line 1469.
Use of uninitialized value in printf at ./Run line 1470.
Use of uninitialized value in printf at ./Run line 1469.
Use of uninitialized value in printf at ./Run line 1470.
Use of uninitialized value in printf at ./Run line 1469.
Use of uninitialized value in printf at ./Run line 1470.
Use of uninitialized value in printf at ./Run line 1721.
Use of uninitialized value in printf at ./Run line 1722.
Use of uninitialized value in printf at ./Run line 1721.
Use of uninitialized value in printf at ./Run line 1722.
Use of uninitialized value in printf at ./Run line 1721.
Use of uninitialized value in printf at ./Run line 1722.
Use of uninitialized value in printf at ./Run line 1721.
Use of uninitialized value in printf at ./Run line 1722.

1 x Dhrystone 2 using register variables  1 2 3 4 5 6 7 8 9 10

1 x Double-Precision Whetstone  1 2 3 4 5 6 7 8 9 10

1 x Execl Throughput  1 2 3

1 x File Copy 1024 bufsize 2000 maxblocks  1 2 3

1 x File Copy 256 bufsize 500 maxblocks  1 2 3

1 x File Copy 4096 bufsize 8000 maxblocks  1 2 3

1 x Pipe Throughput  1 2 3 4 5 6^[[C 7 8 9 10

1 x Pipe-based Context Switching  1 2 3 4 5 6 7 8 9 10

1 x Process Creation  1 2 3

1 x System Call Overhead  1 2 3 4 5 6 7 8 9 10

1 x Shell Scripts (1 concurrent)  1 2 3

1 x Shell Scripts (8 concurrent)  1 2 3

4 x Dhrystone 2 using register variables  1 2 3 4 5 6 7 8 9 10

4 x Double-Precision Whetstone  1 2 3 4 5 6 7 8 9 10

4 x Execl Throughput  1 2 3

4 x File Copy 1024 bufsize 2000 maxblocks  1 2 3

4 x File Copy 256 bufsize 500 maxblocks  1 2 3

4 x File Copy 4096 bufsize 8000 maxblocks  1 2 3

4 x Pipe Throughput  1 2 3 4 5 6 7 8 9 10

4 x Pipe-based Context Switching  1 2 3 4 5 6 7 8 9 10

4 x Process Creation  1 2 3

4 x System Call Overhead  1 2 3 4 5 6 7 8 9 10

4 x Shell Scripts (1 concurrent)  1 2 3

4 x Shell Scripts (8 concurrent)  1 2 3

========================================================================
   BYTE UNIX Benchmarks (Version 5.1.3)

   System: raspberrypi: GNU/Linux
   OS: GNU/Linux -- 4.9.59-v7+ -- #1047 SMP Sun Oct 29 12:19:23 GMT 2017
   Machine: armv7l (unknown)
   Language: en_US.utf8 (charmap="UTF-8", collate="UTF-8")
   CPU 0: ARMv7 Processor rev 4 (v7l) (0.0 bogomips)
          
   CPU 1: ARMv7 Processor rev 4 (v7l) (0.0 bogomips)
          
   CPU 2: ARMv7 Processor rev 4 (v7l) (0.0 bogomips)
          
   CPU 3: ARMv7 Processor rev 4 (v7l) (0.0 bogomips)
          
   12:29:03 up 6 min,  4 users,  load average: 0.16, 0.08, 0.02; runlevel 2018-03-07

------------------------------------------------------------------------
Benchmark Run: Sun Apr 01 2018 12:29:03 - 12:57:08
4 CPUs in system; running 1 parallel copy of tests

Dhrystone 2 using register variables        4333035.5 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     1060.4 MWIPS (9.9 s, 7 samples)
Execl Throughput                                505.3 lps   (29.9 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        142961.6 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           41135.5 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        365027.4 KBps  (30.0 s, 2 samples)
Pipe Throughput                              282493.9 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                  58563.1 lps   (10.0 s, 7 samples)
Process Creation                               2040.7 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   1910.4 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    545.6 lpm   (60.1 s, 2 samples)
System Call Overhead                         572609.5 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0    4333035.5    371.3
Double-Precision Whetstone                       55.0       1060.4    192.8
Execl Throughput                                 43.0        505.3    117.5
File Copy 1024 bufsize 2000 maxblocks          3960.0     142961.6    361.0
File Copy 256 bufsize 500 maxblocks            1655.0      41135.5    248.6
File Copy 4096 bufsize 8000 maxblocks          5800.0     365027.4    629.4
Pipe Throughput                               12440.0     282493.9    227.1
Pipe-based Context Switching                   4000.0      58563.1    146.4
Process Creation                                126.0       2040.7    162.0
Shell Scripts (1 concurrent)                     42.4       1910.4    450.6
Shell Scripts (8 concurrent)                      6.0        545.6    909.3
System Call Overhead                          15000.0     572609.5    381.7
                                                                   ========
System Benchmarks Index Score                                         293.0

------------------------------------------------------------------------
Benchmark Run: Sun Apr 01 2018 12:57:08 - 13:25:34
4 CPUs in system; running 4 parallel copies of tests

Dhrystone 2 using register variables       13303366.2 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     3615.2 MWIPS (11.6 s, 7 samples)
Execl Throughput                               1843.0 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        177911.7 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           49727.3 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        479258.8 KBps  (30.0 s, 2 samples)
Pipe Throughput                              931248.6 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 163404.7 lps   (10.0 s, 7 samples)
Process Creation                               4630.4 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   4356.6 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    529.4 lpm   (60.3 s, 2 samples)
System Call Overhead                        2204195.4 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   13303366.2   1140.0
Double-Precision Whetstone                       55.0       3615.2    657.3
Execl Throughput                                 43.0       1843.0    428.6
File Copy 1024 bufsize 2000 maxblocks          3960.0     177911.7    449.3
File Copy 256 bufsize 500 maxblocks            1655.0      49727.3    300.5
File Copy 4096 bufsize 8000 maxblocks          5800.0     479258.8    826.3
Pipe Throughput                               12440.0     931248.6    748.6
Pipe-based Context Switching                   4000.0     163404.7    408.5
Process Creation                                126.0       4630.4    367.5
Shell Scripts (1 concurrent)                     42.4       4356.6   1027.5
Shell Scripts (8 concurrent)                      6.0        529.4    882.4
System Call Overhead                          15000.0    2204195.4   1469.5
                                                                   ========
System Benchmarks Index Score                                         646.8

```

### 结果说明
测试的结果是一个指数值（index value，如520），这个值是测试系统的测试结果与一个基线系统测试结果比较得到的指数值，这样比原始值更容易得到参考价值，测试集合里面所有的测试得到的指数值结合起来得到整个系统的指数值。


UnixBench也支持多CPU系统的测试，默认的行为是测试两次，第一次是一个进程的测试，第二次是N份测试，N等于CPU个数。这样的设计是为了以下目标：
* 测试系统的单任务性能
* 测试系统的多任务性能
* 测试系统并行处理的能力


