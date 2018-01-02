+++
date = "2017-01-10T12:06:14+08:00"
keywords = ["Reading"]
description = "ReadingDigestsOnServerSideDevelopment"
title = "后台开发读书笔记"
categories = ["Reading"]

+++
从图书馆借回来不少书，其中有一本腾讯工程师写的《后台开发核心技术与应用实践》，这本书的内容很浅显易懂，
基本上涵盖了Linux下C++开发在一般公司能用到的范畴。作者也说了，她写书的初衷在于用最短的篇幅讲解实际后台
用到的核心知识点以便读者能快速进入到实际开发中。扫了扫，前两张用来复习准备面试中有关C++的内容不错。    

想提升的就算了，这本书的代码和调试手段都比较初级，实际工作中，需要更多的借助Google和开源社区来完成。    

这里记录的主要是本人对该书里提到的一些概念的理解.    

### 第一章
#### 函数模板/函数重载
1.2函数章节里，关于函数重载和函数模板的理解可以用下面的代码来解释，左边是用函数重载的情形，可以看到一个
同名函数可以有多个参数版本，而右边的函数模板则引用了模板的概念，大大节约了代码行。     

```
#include<iostream>      					#include<iostream>
using namespace std;    					using namespace std;
int min(int a, int b, int c){   			      |	template<typename T>
							      >	T min(T a,T b,T c){
    if(a>b)a=b; 						    if(a>b)a=b;
    if(a>c)a=c; 						    if(a>c)a=c;
    return a;   						    return a;
}       							}
long long min(long long a,long long b, long long c){          <
    if(a>b)a=b; 					      <
    if(a>c)a=c; 					      <
    return a;   					      <
}       						      <
double min(double a, double b){ //�������������ϵĲ�����ֻ��      <
    if(a-b>(1e-5))a=b;  				      <
    return a;   					      <
}       						      <
int main(){     						int main(){
    int a=1,b=2,c=3;    				      |	   int a=1,b=2,c=3;
    cout<<min(a,b,c)<<endl;     			      |	   cout<<min(a,b,c)<<endl;
    long long a1=100,b1=200,c1=300;     		      |	   long long a1=1000000000,b1=2000000000,c1=3000000000;
    cout<<min(a1,b1,c1)<<endl;  			      |	   cout<<min(a1,b1,c1)<<endl;
    double a2=1.1,b2=2.2;       			      |	   return 0;
    cout<<min(a2,b2)<<endl;     			      <
    return 0;   					      <
}       							}
```
#### 字符数组
字符数组中，关于strlen()和sizeof()可以作为面试中的题目来问面试者。    
#### 函数与指针
函数指针的情形, 注意这里对函数的引用是直接将函数的地址赋给f.    

```
#include<iostream>
using namespace std;
int Mmin(int x,int y){
     if(x<y)return x;
     return y;
}
int Mmax(int x,int y){
    if(x>y)return x;
    return y;
}
int main(){
    int (*f)(int x,int y);
    int a=10,b=20;
    f=Mmin;   //��Mmin���������ڵ�ַ����f
    cout << (*f)(a,b)<<endl;
    f=Mmax;  //��Mmax���������ڵ�ַ����f
    cout << (*f)(a,b)<<endl;
    return 0;
}
```
#### 结构体/共用体/枚举
共用体的定义可以回忆一下：用关键字union来定义，是一种特殊的类。在一个共用体里可以定义多种
不同的数据类型，这些数据类型共享一段内存，在不同的时间里保存不同的数据类型和长度的变量。但是，
同一时间内只能存储一种类型的数据。其存在的目的是为了节省空间。     

判断大小端的程序有点意思:    

```
#include<iostream>
using namespace std;
union TEST{
    short a;
    char b[sizeof(short)];
};
int main(){
    TEST test;
    test.a=0x0102;// �������ù�����������ֻ�����ù����������еĳ�Ա��
    if(test.b[0]==0x01&&test.b[1]==0x02){
        cout<<"big endian."<<endl;
    }
    else if(test.b[0]==0x02&&test.b[1]==0x01){
        cout<<"small endian."<<endl;
    }
    else{
        cout<<"unknown"<<endl;
    }
    return 0;
}
```
枚举要注意类似于下面的题目:    

```
enum fruits{apple=3,orange,banana=7,bear};
结果为: 3, 4, 7, 8
```
#### 占用字节数的计算
鉴于这个题材经常在面试中被问到，单独拎出来写一段：    

Union：     

```
#include<iostream>
using namespace std; 
union A{
   int a[5];
   char b;
   double c;
};
int main(){
   cout<<sizeof(A)<<endl;
   return 0;
}
```
最长的double(8Byte)对齐，因而占用的大小应该是`int(4Byte)*5=20`, 而8字节对齐应该是3x8=24。因而运行结果为24


Struct:    

```
#include<iostream>
using namespace std; 
struct B{
  char a;
  double b;
  int c;
}test_struct_b;
int main(){
   cout<<sizeof(test_struct_b)<<endl;
   return 0;
}
```
计算方法是：     

```
char a , 1
补充7
double b, 8
int c, 4
补充4
```
所以结果为1+7+8+4+4=24.     

混合结构体:      

```
#include<iostream>
using namespace std;
typedef union{
    long i;
    int k[5];
    char c;
} UDATE; 
struct data{
    int cat;
    UDATE cow;
    double dog;
}too; 
UDATE temp; 
int main(){
    cout<<sizeof(struct data)+sizeof(temp)<<endl;
    return 0;
}
```
原书有错：   

sizeof(temp)大小为24, 因为temp是UDATA类型结构。     
sizeof(struct data)的计算则是结构体大小变化，每个变量需要占据各自独立的空间，依次为:    

```
int cat: 4
4字节填充(参照double 8字节对齐)
UPDATE cow: 24
double dog: 8
一共为: 4+4+24+8=40
```
综上所述，结果为40+24=64.    

#### do...while(0)的使用
这个问题可以在面试的时候问面试者。    

例程:    

```
# define Foo(x) do{\
    statement one;\
    statement two;\
}while(0)
```
在宏替换时这样的写法不会破坏以下的情形:     

```
if(condition)
    statement one;
    statement two;
else
    //.....
```
如果去掉do...while(0)则会导致else语句孤立而出现编译错误，加了以后，则使得宏展开后，仍然保留了
原始的语义，从而保证程序的正确性。   
#### extern "C"
加这个是为了让编译器将其当成C语言处理。    

```
#ifdef __cpluscplus
    extern "C" {
#endif 
```

### 第二章
#### 类与结构体
定义的异同:    

```
class CStudent{         				      |	struct SStudent{
    int num;    					      |	public:
    char name[20];      				      <
    int age;           //��Щ�����ݳ�Ա��Ҳ��Ϊ��Ա����             <
    void display(){   //���ǳ�Ա����      			    void display(){   //���ǳ�Ա����
        cout<<"num:"<<num<<endl;				        cout<<"num:"<<num<<endl;
        cout<<"name:"<<name<<endl;      			        cout<<"name:"<<name<<endl;
        cout<<"age:"<<age<<endl;				        cout<<"age:"<<age<<endl;
    }   						      |	    }                  //����û�зֺ�
};      						      |	private:
CStudent cstu1,cstu2;//������2������    		      |	    int num;
							      >	    char name[20];
							      >	    int age;
							      >	};

```
#### 类的封装性
源代码如下:    

```
➜  0205 cat student.h
class CStudent{
public:
    void display();
private:
    int num;
    char name[20];
    int age;
};
➜  0205 cat main.cpp 
#include<iostream>
#include "student.h" //ע��������˫����
int main(){
    CStudent stu1;//����stu1����
    stu1.display();//ָ��stu1�����ĳ�Ա����
    return 0;
}
➜  0205 cat student.cpp 
#include<iostream>
#include "student.h" //������Ҫinclude����ͷ�ļ��������޷��ҵ�Student��
using namespace std; 
void CStudent::display(){  //����Ҫע����Student����
    cout<<"num:"<<num<<endl;
    cout<<"name:"<<name<<endl;
    cout<<"age:"<<age<<endl;
}
``` 
运行结果:    

```
num:4196688
name:
age:0
```
这样的结果是因为没有对数据成员进行初始化而导致的。    

#### 构造函数
更改上面的代码:    

```
➜  0205_1 cat student.h
class CStudent{
public:
    CStudent() {
      num = 0;
      age = 0;
    }
    void display();
private:
    int num;
    char name[20];
    int age;
};
➜  0205_1 ./test 
num:0
name:
age:0
```
#### 静态数据成员
例子:    

```
➜  chapter02 cat ./0212.cpp 
#include<iostream>
using namespace std;
class Base{
public:
    static int var;
};
int Base::var=10;
class Derived:public Base{
};
int main(){
    Base base1;
    base1.var++;//ͨ������������
    cout<<base1.var<<endl;//����11
    Base base2;
    base2.var++;
    cout<<base2.var<<endl;//����12
    Derived derived1;
    derived1.var++; 
    cout<<derived1.var<<endl;//����13
    Base::var++;//ͨ����������
    cout<<derived1.var<<endl;//����14
    return 0;
}
➜  chapter02 ./0212
11
12
13
14
```
#### 对象存储空间
这个和第一章的存储空间可以结合起来看。    

空类的存储空间为1:    

```
➜  chapter02 cat 0214.cpp 
#include<iostream>
using namespace std;
class CBox{
};
int main(){
    CBox boxobj;
    cout<<sizeof(boxobj)<<endl;//����1
    return 0;
}
➜  chapter02 ./0214
1
```
有成员变量的类的存储空间:     

```
➜  chapter02 cat 0215.cpp 
#include<iostream>
using namespace std;
class CBox{
    int length,width,height;
};
int main(){
    CBox boxobj;
    cout<<sizeof(boxobj)<<endl;
    return 0;
}
➜  chapter02 ./0215
12
```
有静态成员变量时，静态成员变量不占据对象的内存空间:    

```
➜  chapter02 cat 0216.cpp 
#include<iostream>
using namespace std;
class CBox{
    int length,width,height;
    static int count;
};
int main(){
    CBox boxobj;
    cout<<sizeof(boxobj)<<endl;
    return 0;
}
➜  chapter02 ./0216
12
```
成员函数不占据空间:    

```
➜  chapter02 cat 0217.cpp 
#include<iostream>
using namespace std;
class CBox{
    int foo();
};
int main(){
    CBox boxobj;
    cout<<sizeof(boxobj)<<endl;
    return 0;
}
➜  chapter02 ./0217
1
```
构造函数与析构函数也不占据空间:    

```
class CBox{
public:
    CBox(){};
    ~CBox(){};
};
大小为1
```

虚析构函数，占用大小为8:     

```
class CBox{
public:
    CBox(){};
    virtual ~CBox(){};
};
大小为8
```
#### 类模板
操作整数的类与操作浮点数的类:    

```
class Operation_int{    				      |	class Operation_double{
public: 							public:
    Operation_int(int a,int b):x(a),y(b){}      	      |	    Operation_double(double a, double b):x(a),y(b){}
    int add(){  					      |	    double add(){
        return x+y;     					        return x+y;
    }   							    }
    int subtract(){     				      |	    double subtract(){
        return x-y;     					        return x-y;
    }   							    }
private:							private:
    int x,y;    					      |	    double x,y;
};      							};

```
用类模板来抽象:    

```
#include <iostream>
using namespace std;
template<class T>//����һ��ģ�壬����������ΪT
class Operation {
public:
    Operation (T a, T b):x(a),y(b){}
    T add(){
        return x+y;
    }
    T subtract(){
        return x-y;
    }
private:
    T x,y;
};
int main(){
    Operation <int> op_int(1,2);
    cout<<op_int.add()<<" "<<op_int.subtract()<<endl;//����3��-1
    Operation <double> op_double(1.2,2.3);
    cout<<op_double.add()<<" "<<op_double.subtract()<<endl;//����3.5��-1.1
    return 0;
}
➜  chapter02 ./0224
3 -1
3.5 -1.1
```

#### 虚函数/纯虚函数
虚函数可以使得基类指针访问派生类中的同名函数:    
而纯虚函数则是因为：用基类本身生成对象不合情理，例如用动物作为基类来抽象具体的动物，而动物这个基类
本身是不能被实例化的。    

析构函数不是虚函数时，容易引发内存泄漏:    

```
➜  chapter02 ./0231
Base::Base()
Derive::Derive()
Base::~Base()
➜  chapter02 cat 0231.cpp 
#include<iostream>
using namespace std;
class Base{
public:
    Base(){ std::cout<<"Base::Base()"<<std::endl; }
    ~Base(){ std::cout<<"Base::~Base()"<<std::endl; }
};
class Derive:public Base{
public:
    Derive(){ std::cout<<"Derive::Derive()"<<std::endl; }
    ~Derive(){ std::cout<<"Derive::~Derive()"<<std::endl; }
};
int main(){
    Base* pBase = new Derive(); 
    /*����base classed������Ŀ����Ϊ������"ͨ��base class�ӿڴ���derived class����"*/
    delete pBase;
    return 0;
}
```
这是由C++的定义指出的：如果一个派生类对象经由一个基类指针被删除，而该基类带有一个非虚析构函数，则会导致
派生类中的成分没被销毁。    

所以，需要把析构函数定义为虚函数。    

```
➜  chapter02 ./0232
Base::Base()
Derive::Derive()
Derive::~Derive()
Base::~Base()
➜  chapter02 cat 0232.cpp 
#include<iostream>
using namespace std;
class Base{
public:
    Base(){ std::cout<<"Base::Base()"<<std::endl; }
    virtual ~Base(){ std::cout<<"Base::~Base()"<<std::endl; }
};

class Derive:public Base{
public:
    Derive(){ std::cout<<"Derive::Derive()"<<std::endl; }
    ~Derive(){ std::cout<<"Derive::~Derive()"<<std::endl; }
};

int main(){
    Base* pBase = new Derive();
    delete pBase;
    return 0;
}
```

#### 单例模式
理解：一台计算机上可以连好几台打印机，但是打印程序只能有一个，这里就可以通过单例模式来避免两个打印作业
被同时输出到打印机中。    

```
➜  chapter02 ./0233
s1=s2
➜  chapter02 cat 0233.cpp 
#include<iostream>
using namespace std;
class CSingleton{
private:
	CSingleton(){   //构造函数是私有的
	}
	static CSingleton *m_pInstance;
public:
	static CSingleton * GetInstance(){
        if(m_pInstance == NULL)  //判断是否是第一次调用
	        m_pInstance = new CSingleton();
	    return m_pInstance;
	}
};
CSingleton * CSingleton::m_pInstance=NULL;//初始化静态成员变量
int main(){
    CSingleton *s1= CSingleton::GetInstance();
    CSingleton *s2= CSingleton::GetInstance();
    if(s1==s2){
        cout<<"s1=s2"<<endl; 
    }
    return 0;
}
```
单例类的特点：    

```
1. 有指向唯一实例的静态指针m_pInstance，且为私有。   
2. 有一个公有函数用于获取唯一的实例。
3. 构造函数是私有的，不能在别处创建该类的实例.
```
