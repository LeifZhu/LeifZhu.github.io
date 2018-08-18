---
title: 解决error:&ensp; Your local changes to the following files would be overwritten by merge
date: 2018-08-18 23:42:16
tags: [git]
---

本文介绍决执行`git pull`时，错误`error: Your local changes to the following files would be overwritten by merge`的产生原因和解决方法。
<!--more-->
## 问题：git pull时遇到冲突
git pull博客源代码仓库的时候发现冲突

```
[Leif@pc90005 blog]$ git pull
Updating f89b7be..01cc068
error: Your local changes to the following files would be overwritten by merge:
	_config.yml
Please commit your changes or stash them before you merge.
Aborting
```
执行git status检查目录状态显示如下

```
[Leif@pc90005 blog]$ git status
On branch src
Your branch is behind 'origin/src' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   _config.yml

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	source/_posts/MXNet-alexnet-cifar10.md
	source/_posts/MXNet-stop-training-by-loss.md
	source/_posts/git-submodule.md
	source/_posts/ssh-collection.md

no changes added to commit (use "git add" and/or "git commit -a")

```
## 原因
从上面的结果来看，是因为我修改了\_config.yml，导致该文件和服务器上最新的版本不同导致的合并冲突。此时如果git将服务器上的\_config.yml（为了区分，记为\_config.yml[remote]）拉下来直接覆盖本地文件（记为\_config.yml[local]），那么\_config.yml[local]中所做的更改就再也找不回来了，这是因为\_config.yml[local]还没有commit。git不会擅做这种主张，导致你没有后悔药吃。

## 解决方案
### checkout:丢弃本地所做的更改
```
git checkout HEAD _config.yml
```
此时执行`git pull`， git发现当前工作区是干净的，就放心地将_config.yml[remote]下拉回来了。注意，这样的话，你等于主动丢弃\_config.yml[local]中所做的更改，这些更改无法恢复。
### commit: 将修改提交到仓库
```
git commit -a -m "modified _config.yml"
```
同样地，这时工作区是干净的，所以`git pull`也没有问题。因为已经提交到仓库，你总是有办法找回_config.yml[local]。
### stash: 将当前的工作目录另存到储藏栈上
如果你不想丢失\_config.yml[local]又不想因为这么一点小小的改动就创建一次commit（比如，你希望至少完成一篇博客才做一次提交），那么可以使用`git stash`。按序执行以下三条命令可以让你将\_config.yml[remote]下拉回来，然后和\_config.yml[local]合并

```
git stash
git pull
git stash pop
```
关于stash的功能和用法可以参考[[1]](https://git-scm.com/book/zh/v2/Git-工具-储藏与清理)。简单来说，`git stash`会

1. 先将你当前的工作目录暂存到一个“储藏栈”的栈顶（这个储藏栈和“暂存区”或者“仓库”不是一个地方）
2. 然后恢复到你上次提交完的状态(相当于再执行`git reset --hard`)

而`git stash pop`则会

1. “储藏栈”栈顶的一次暂存和当前工作目录中文件合并
2. 然后删除储藏栈顶的暂存。

## 参考
[1]https://git-scm.com/book/zh/v2/Git-工具-储藏与清理