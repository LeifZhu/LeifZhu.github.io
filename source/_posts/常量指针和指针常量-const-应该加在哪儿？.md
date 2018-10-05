---
title: 常量指针和指针常量--const 应该加在哪儿？
date: 2018-10-05 14:05:09
tags: [C++]
category: 人生经验
---

常量指针和指针常量傻傻分不清楚？ 教你记住怎么加const关键字。
<!--more-->
这里我不想再介绍常量指针和指针常量的定义，拗口的定义是导致我总是记不住const应该怎么加的罪魁祸首。下面只说const怎么使用。  

## 结论
1. `int const * ptr`和`const int * ptr` 是一个意思，就是你不能通过`*ptr = 1`给ptr指向的变量赋值。
2. `int * const ptr`的意思是， 只能在ptr初始化时确定它指向的变量， 此后不能再通过`ptr = anotherPtr`的方式改变ptr的指向。
3. 一种复合的情况是 `const int* const ptr`, 意思是二者都不能变。

## 怎么记?
把typename去掉（比如上面的`int`），然后看剩下的部分。
1. `const * ptr`的意思是`*ptr`不能变，也就是指针指向的对象不能变；
2. `* const ptr`的意思是`ptr` 不能变，也就是指针不能变。
