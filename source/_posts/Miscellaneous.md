---
title: Miscellaneous
date: 2018-08-21 22:17:45
tags: [linux, deep learning, git, apt]
category: 见得多了
---

This post will be used to keep track of daily bugs, workaroud, solutions, collections,links and so on.
<!--more-->

## git
官方文档是坠吼的：[English](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control)
---[中文](https://git-scm.com/book/zh/v2)
### .gitignore快速教程
一个例子,来自[这里](https://www.cnblogs.com/ShaYeBlog/p/5355951.html)
```
# 此为注释 – 将被 Git 忽略
 
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```
### 解决.gitignore不生效
把暂存区缓存清除
```
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

--------------------------------------------
## apt/apt-get

### add-apt-repository后apt update报错
```
E: The repository 'http://ppa.launchpad.net/gummi/gummi/ubuntu bionic Release' does not have a Release file
```
解决方法在[这里](https://www.reddit.com/r/Ubuntu/comments/8gcja7/what_should_i_do_if_a_ppa_does_not_have_a_release/)
```
sudo add-apt-repository --remove ppa:gummi/gummi
sudo apt update
sudo apt install gummi
```
> Never add a PPA if you don't have to.


-------------------------------------------
## 深度学习

### 用Alexnet在cifar10上训练报错
Alexnet的第一层的卷积核以及步长是多大？过完两层之后特征图还有多大？如果你一定要在cifar10上训练Alexnet, 要么在预处理时把图放大一下尺寸，要么修改Alexnet各层的卷积核和池化窗口大小。


----------------------------------------------
## Tensorflow

### tensoflow low level api 
[click here](https://www.tensorflow.org/guide/low_level_intro)

有关tensorflow的一些基础概念以及低级的api都在这里。只使用高级api可能永远不知道模型里面发生了什么，还有就是有时候真的需要更低级的api，复杂意味着灵活，简单意味着死板。


