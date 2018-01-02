+++
date = "2017-03-09T15:55:10+08:00"
description = "Linux"
title = "Reading Digest On Programming Interview"
categories = ["Linux"]
keywords = ["Linux"]

+++
最近扫了扫一本知名书《程序员面试宝典》。众所周知国内的面试都得要靠扫书来的嘛。    

然而这本书上的有些习题，答案也未必为对，这里列举一个:    

### CPU架构导致的sizeof
原题目如下:    

```
#include <stdio.h>
#include <stdlib.h>
struct {
	short a1;
	short a2;
	short a3;
}A;
struct {
	long a1;
	short a2;
}B;

int main(void)
{
	char *ss1= "0123456789";
	char ss2[]= "0123456789";
	char ss3[100] = "0123456789";
	int ss4[100];
	char q1[]="abc";
	char q2[]="a\n";
	char *str1= (char*)malloc(100);
	void *str2= (void*)malloc(100);

	printf("Sizeof ss1 is %d\n", sizeof(ss1));
	printf("Sizeof ss2 is %d\n", sizeof(ss2));
	printf("Sizeof ss3 is %d\n", sizeof(ss3));
	printf("Sizeof ss4 is %d\n", sizeof(ss4));
	printf("Sizeof q1 is %d\n", sizeof(q1));
	printf("Sizeof q2 is %d\n", sizeof(q2));
	printf("Sizeof str1 is %d\n", sizeof(str1));
	printf("Sizeof str2 is %d\n", sizeof(str2));
	printf("Sizeof A is %d\n", sizeof(A));
	printf("Sizeof B is %d\n", sizeof(B));

	return 0;
}
```
在我的机器上的答案是:    

```
Sizeof ss1 is 8
Sizeof ss2 is 11
Sizeof ss3 is 100
Sizeof ss4 is 400
Sizeof q1 is 4
Sizeof q2 is 3
Sizeof str1 is 8
Sizeof str2 is 8
Sizeof A is 6
Sizeof B is 16
```
而原文答案则有差距，例如ss1的大小为4， str/str2的结果为4， B的大小为8.
为什么？     

我猜想是因为CPU架构的不同，64位系统与32位系统的不同而导致。     

为了验证，我创建了一台virtualbox的32位Ubuntu16.04的机器， 验证结果:    

```
root@ubuntu:~# uname -a
Linux ubuntu 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:06:14 UTC 2016 i686 i686 i686 GNU/Linux
root@ubuntu:~# ./test 
Sizeof ss1 is 4
Sizeof ss2 is 11
Sizeof ss3 is 100
Sizeof ss4 is 400
Sizeof q1 is 4
Sizeof q2 is 3
Sizeof str1 is 4
Sizeof str2 is 4
Sizeof A is 6
Sizeof B is 8
```
由此可见，其答案局限于32位平台，不能单纯的盲从于参考答案。    

### sizeof(string)变化
例子：    

```
#include <iostream>
#include <string>
using namespace std;

int main()
{
	string strArr1[] = {"Trend", "Micro", "Soft"};
	string *pStrArr1 = new string[2];
	pStrArr1[0] = "US";
	pStrArr1[1] = "CN";

	for(int i = 0; i<sizeof(strArr1)/sizeof(string); i++)
		cout<<strArr1[i];
	for(int j = 0; j<sizeof(pStrArr1)/sizeof(string); j++)
		cout<<pStrArr1[j];

	cout<<endl;
	cout<<sizeof(pStrArr1)<<endl;
	cout<<sizeof(string)<<endl;
	return 0;
}
```
实际的输出应该为:    

```
TrendMicroSoft
8
32
```
这是因为库函数发生了变化，sizeof(string)的大小已经调整为32, 因而书里说的TrendMicroSoftUS是永远不会打印出来的。     
