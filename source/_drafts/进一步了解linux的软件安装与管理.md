---
title: 进一步了解linux的软件安装与管理 
date: 2018-09-15 17:47:59
tags: [linux]
category: 微小贡献
---

这个问题可能是一个linux用户面对的最基础的问题之一。在linux上安装软件有时是一件轻而易举的事情(使用apt或yum这样的[Package Mangement](https://en.wikipedia.org/wiki/Package_manager) System)，有时却又异常困难(当你需要用从.deb、.rpm安装包安装，甚至编译源码安装时)。本文介绍这三种方式安装软件的套路以及原理。

<!--more-->
## 是什么导致了软件安装的困难性？
动态链接库是导致软件安装困难的根本原因是动态链接库依赖(linux下的shared object, 通常以.so结尾，参考[shared library](https://en.wikipedia.org/wiki/Library_(computing)#Shared_libraries),如果你不知道代码编译和运行中的linking机制,可以参考[CSAPP](http://csapp.cs.cmu.edu/3e/students.html)的第七章)。比如你要安装的软件t可能依赖a.so和b.so,而a.so与b.so又有可能分别依赖于c.so与d.so。在实际中遇到的情况甚至还要更复杂，这种复杂的依赖关系被称为[Dependency hell](https://en.wikipedia.org/wiki/Dependency_hell)

## 在linux上安装软件的三种方法
要安装一个软件，我们通常有三种方式:
1. 使用package management system (如apt和yum)
2. 下载.rpm或者.deb安装包安装
3. 从源码编译安装
尽管（大多数时候）在使用时，我们的优先级是1>2>3，但是为了叙述方便，我决定按照3->2->1的顺序介绍
### 最原始的方式：从源码编译安装


### 折中方案：从.rpm或者.deb安装

### 最喜闻乐见的方式：使用package management sytem

### 总结

## 当你必须手动解决依赖时有哪些策略

使用软连接
find
strings

apt fix

安装依赖

## 其它
1. 一个约定俗成的规矩源码安装的软件通常放在/usr/local/目录下
2. apt和apt-get的关系



```
leizhu@pc-office:~$ sudo apt install mailspring
[sudo] password for leizhu: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package mailspring

```

```
leizhu@pc-office:~/Downloads$ sudo dpkg -i mailspring-1.4.2-amd64.deb 
[sudo] password for leizhu: 
Selecting previously unselected package mailspring.
(Reading database ... 409752 files and directories currently installed.)
Preparing to unpack mailspring-1.4.2-amd64.deb ...
Unpacking mailspring (1.4.2) ...
dpkg: dependency problems prevent configuration of mailspring:
 mailspring depends on libsecret-1-dev; however:
  Package libsecret-1-dev is not installed.
 mailspring depends on gir1.2-gnomekeyring-1.0; however:
  Package gir1.2-gnomekeyring-1.0 is not installed.

dpkg: error processing package mailspring (--install):
 dependency problems - leaving unconfigured
Processing triggers for hicolor-icon-theme (0.17-2) ...
Processing triggers for gnome-menus (3.13.3-11ubuntu1.1) ...
Processing triggers for desktop-file-utils (0.23-1ubuntu3.18.04.1) ...
Processing triggers for mime-support (3.60ubuntu1) ...
Errors were encountered while processing:
 mailspring

```


```
leizhu@pc-office:~/Desktop/blog/source$ sudo apt --fix-broken install
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Correcting dependencies... Done
The following additional packages will be installed:
  gir1.2-gnomekeyring-1.0 libsecret-1-dev
The following NEW packages will be installed:
  gir1.2-gnomekeyring-1.0 libsecret-1-dev
0 upgraded, 2 newly installed, 0 to remove and 65 not upgraded.
1 not fully installed or removed.
Need to get 182 kB of archives.
After this operation, 2,268 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://hk.archive.ubuntu.com/ubuntu bionic/main amd64 libsecret-1-dev amd64 0.18.6-1 [176 kB]
Get:2 http://hk.archive.ubuntu.com/ubuntu bionic/universe amd64 gir1.2-gnomekeyring-1.0 amd64 3.12.0-1build1 [5,946 B]
Fetched 182 kB in 1s (214 kB/s)                     
Selecting previously unselected package libsecret-1-dev:amd64.
(Reading database ... 409989 files and directories currently installed.)
Preparing to unpack .../libsecret-1-dev_0.18.6-1_amd64.deb ...
Unpacking libsecret-1-dev:amd64 (0.18.6-1) ...
Selecting previously unselected package gir1.2-gnomekeyring-1.0.
Preparing to unpack .../gir1.2-gnomekeyring-1.0_3.12.0-1build1_amd64.deb ...
Unpacking gir1.2-gnomekeyring-1.0 (3.12.0-1build1) ...
Setting up libsecret-1-dev:amd64 (0.18.6-1) ...
Setting up gir1.2-gnomekeyring-1.0 (3.12.0-1build1) ...
Setting up mailspring (1.4.2) ...
```