---
title: Solve "libstdc++.so.6:&ensp;version CXXABI_1.3.8 not found"
date: 2018-08-24 00:13:17
tags: [gcc, linux]
category: 人生经验
---

`libstdc++.so.6:version CXXABI_1.3.8 not found`错误出现的原因是libstdc++.so.6的版本太旧。这个问题在CentOS中尤其容易出现, 因为目前yum的源中维护的gcc版本比较旧(4.8.5)。这个问题可以通过安装较新版本的gcc解决。
<!--more-->
## 安装gcc
如果你的系统支持用包管理工具(比如apt)安装新版gcc，那么问题就已经解决了；如果不支持（比如在CentOS上），那么就源码安装gcc吧。

1. 从https://ftp.gnu.org/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz下载安装包
2. 依次执行下面的命令

```
tar -xvf gcc-7.3.0.tar.gz
cd gcc-7.3.0
yum install libmpc-devel mpfr-devel gmp-devel
./configure --with-system-zlib --disable-multilib --enable-languages=c,c++
make -j 8
make install
gcc --version
```
完事记得检查新的libstdc++.so.6在哪里

```
find / -name libstdc++.so.6
```
将该目录加入环境变量`LD_LIBRARY_PATH`(一般是`/usr/local/lib`或`/usr/local/lib64`里面那个，如果不确定的话用`strings /path/to/libstdc++.so | grep CXXABI_1.3.8`检查一下)

## 注意事项：`devtoolset-x`解决不了你的问题
如果你是用`devtoolset-x`安装的gcc，那么`libstdc++.so.6:version CXXABI_1.3.8 not found`还是解决不了，因为
> the devtoolset-x packages actually just wrap the standard system libstdc++.so

参考这里[[1]](https://stackoverflow.com/questions/46172600/usr-lib64-libstdc-so-6-version-cxxabi-1-3-8-not-found)。

## 参考
[1]https://stackoverflow.com/questions/46172600/usr-lib64-libstdc-so-6-version-cxxabi-1-3-8-not-found

