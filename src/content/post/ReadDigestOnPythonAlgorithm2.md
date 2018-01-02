+++
categories = ["Algorithm"]
date = "2016-09-27T09:41:02+08:00"
description = "Read digest on python algorithm"
keywords = ["Algorithm"]
title = "ReadDigestOnPythonAlgorithm2"

+++
第三章笔记。   
### 目的

```
* 理解抽象数据结构，stack,queue,dequeue,list.    
* 使用python lists实现抽象数据结构queue, deque.    
* 理解实现基本线性数据结构的性能.    
* 理解prefix,infix,postfix.    
* 使用stack陪你国家postfix表达式.    
* 使用stack来将表达式从infix到postfix.    
* 使用队列模拟基本计时模拟器
* 何种情况下该使用何种数据结构.
* 使用node/参考模式实现抽象数据结构(链表).    
* 对比我们实现的链表和Python的链表实现的性能.   
```
### 线性数据结构
线性数据结构包括: 栈，队列，双端队列，列表。    
线性是因为存在两端，左/右，或者前/后，或者顶/底。    
线性数据结构的差别在于数据添加/删除的方式。线性数据结构可以组合来解决很多难题。

### Stack
LIFO， Last In, First Out. 先进后出，后进先出。   
Stack定义的抽象数据结构，见原文。    

```
class Stack:
     def __init__(self):
         self.items = []

     def isEmpty(self):
         return self.items == []

     def push(self, item):
         self.items.append(item)

     def pop(self):
         return self.items.pop()

     def peek(self):
         return self.items[len(self.items)-1]

     def size(self):
         return len(self.items)

s=Stack()

print(s.isEmpty())
s.push(4)
s.push('dog')
print(s.peek())
s.push(True)
print(s.size())
print(s.isEmpty())
s.push(8.4)
print(s.pop())
print(s.pop())
print(s.size())
```
运行结果:    

```
$ python2 stack.py 
True
dog
3
False
8.4
True
2
```

