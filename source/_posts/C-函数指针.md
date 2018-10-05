---
title: C++函数指针
tags:
  - C++
category: 人生经验
date: 2018-10-05 13:11:53
---


C++的函数指针算是一个老生常谈的问题，网上有很多相关内容，这里主要指出关于声明和调用的的方法。
<!--more-->

## 声明时的括号必不可少
假如你要声明一个名为`funcP`的函数指针，这个指针指向的函数有两个`int`参数， 函数的返回值是`bool`，那么你的声明应该为
```C++
bool (*funcP)(int, int);
```
**注意**: 这里`(*funcP)`两侧的括号必不可少。为什么？想一想如果去掉这个括号你得到的是什么？（是的，一个返回bool指针的函数的声明。）

## 赋值和调用时的多种形式
假如你有一个原型如下的函数
```C++
bool compare(int, int);
```
现在你想让上面的函数指针指向该函数，那么以下两种方式都是合法的
```C++
// 下面的第一种方式说明函数名"compare"就是一个函数指针（常量)，
// 这个和数组a[10]中的a是一个指针常量类似
funcP = compare; 

// 以下的第二种方式也被允许，并且这样显式地表达出funcP是一个指针，
// 使得函数指针的使用与普通指针一致
funcP = &compare;
```
不论你使用何种方式赋值, 在通过函数指针`funcP`调用函数时，下面的两种方式也都合法
```C++
funcP(4, 5);
(*funcP)(4, 5);
```
### 函数指针数组(Array of function pointers)
声明方法
```C++
bool (*funcPointerArray[10])(int, int);
```
调用方法
```
funcPointerArray[0](2, 3);
(*(funcPointerArray[0]))(2, 3);
```

### 一个例子
最后用一个例子结束本篇博客。

```C++
// functionPtr.cpp

#include <iostream>

bool compare(int a, int b)
{
	return a > b;
}

int main()
{
	bool (*funcPointerArray[2])(int, int); // array of function pointer 
	funcPointerArray[0] = compare; // assignment method 1
	funcPointerArray[1] = &compare; // assignment method 2

	// no matter which assignment method you used
	// you can use both two calling method to call the function
	// the pointer points to
	std::cout<<"using assignment method 1 and calling method 1:"<<std::endl;
	std::cout<<"5>4? "<<funcPointerArray[0](5, 4)<<std::endl;

	std::cout<<"using assignment method 1 and calling method 2:"<<std::endl;
	std::cout<<"5>4? "<<(*funcPointerArray[0])(5, 4)<<std::endl;
	
	std::cout<<"using assignment method 2 and calling method 1:"<<std::endl;
	std::cout<<"5>4? "<<funcPointerArray[1](5, 4)<<std::endl;
	
	std::cout<<"using assignment method 2 and calling method 2:"<<std::endl;
	std::cout<<"5>4? "<<(*funcPointerArray[1])(5, 4)<<std::endl;

	return 0;
}

```
执行结果：
```
leizhu@pc-office:~/git_repo/examples$ ./functionPtr 
using assignment method 1 and calling method 1:
5>4? 1
using assignment method 1 and calling method 2:
5>4? 1
using assignment method 2 and calling method 1:
5>4? 1
using assignment method 2 and calling method 2:
5>4? 1
```

### 参考
[这篇博客](https://www.cnblogs.com/windlaughing/archive/2013/04/10/3012012.html)内容更加详细。
