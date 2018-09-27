---
title: “cuda编程入门”
tags: [cuda]
category: 历史进程
---

这篇博客持续更新最近入门cuda编程的一些进展和遇到的重点。
<!--More-->
## cuda是什么

## cuda安装
我是通过runfile安装的，下载地址在[这里](https://developer.nvidia.com/cuda-downloads)。
我在安装过程中主要参考了下面两个材料:
* [官方文档(英文版)](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile-nouveau)
* [一篇博客(中文版)](https://blog.csdn.net/QLULIBIN/article/details/78714596)

上面的博客大部分是官方文档的翻译，也包括一些作者的总结。我按官方文档执行了Pre-installation Actions和Deisabling Nauveou，然后执行runfile， 接着就成功编译了示例代码。这期间出现了一些library缺失的问题，谷歌一下能很快解决。

根据我的安装过程来看，最重要的一步是**禁用Nauveou然后重启linux**继续安装。

## 学习cuda编程
主要参考资料: [Intro to Parallel Programming](https://www.youtube.com/watch?v=F620ommtjqk&list=PLAwxTw4SYaPnFKojVQrmyOGFCqHTxfdv2)
课程的slide和代码练习可以在github上找到: [Udacity CS344](https://github.com/udacity/cs344)

## Problem shooting
### 为什么cudaMalloc中第一个参数是void**类型，而不是单指针？
提示：传值？传引用？
[这个链接](https://devtalk.nvidia.com/default/topic/418460/-void-amp-d-why-two-/)中_teju_给出了很好的回答(5楼)。

### cmake编译cuda?
参考[这里](https://www.cnblogs.com/haiyang21/p/7788063.html).



