---
title: 解决Hexo博客主题不能多机同步的问题:&ensp; git submodule
date: 2018-08-14 23:38:46
tags: [git, hexo]
---
* 当git仓库R下有子目录S也是一个git仓库时，在R下执行`git push`不会推送R包含的内容
* 因为上述原因，放在`blog/theme/`下的主题文件夹可能不能用github你的多台机器之间同步
* 如果你希望解决上一条中的问题，可以使用git submodule ([[1]](https://git-scm.com/book/en/v2/Git-Tools-Submodules), [[2]](https://chrisjean.com/git-submodules-adding-using-removing-and-updating/), [[3]](https://gist.github.com/myusuf3/7f645819ded92bda6677))
* 相比于使用网盘同步博客源码仓库或者删掉主题中的.git文件夹，用git submodule管理hexo主题的好处是，当该主题更新时，你可以就地更新到本地而不用需要从头开始配置个人数据。

留恢复现场的坑待填。

## 与submodule有关的命令

```
git clone --recursive 
```
```
git submodule add
```
```
git submodule init
```
```
git submodule update
```

## 参考

[1] https://git-scm.com/book/en/v2/Git-Tools-Submodules

[2] https://chrisjean.com/git-submodules-adding-using-removing-and-updating/

[3] https://gist.github.com/myusuf3/7f645819ded92bda6677


