---
title: Miscellaneous
date: 2018-08-21 22:17:45
tags: [linux, deep learning, C++]
category: 见得多了
---

This post will be used to keep track of daily bugs, workaroud, solutions, collections,links and so on.
<!--more-->

## linux
### inode
两个很好的参考链接：
* [阮一峰的日志](http://www.ruanyifeng.com/blog/2011/12/inode.html)
* [鸟哥的linux私房菜](http://cn.linux.vbird.org/linux_basic/0230filesystem.php)

概括：
* 磁盘分为inode区和data block区，磁盘一经格式化就会少一部分空间，这部分空间被inode区占用。尽管**分配时是按磁盘块比例分配**的，在后续使用中实际上是**一个inode对应一个文件，一个文件可能对应多个inode**, 因既有可能出现inode用完而data block还有空余空间的情况(每个文件都太小，比如，只占一个block)，也有可能出现inode还剩很多，但data block已经装满的情况(每个文件都很大)。
* 文件目录也是文件，它有inode，也有data blocks，它的data blocks里以[\<inode\> \<filename\>}]的格式记录该目录下文件名->inode号的映射
* inode里面除了data block的索引区块（分为直接索引，一次间址， 两次间址， 三次间址）外，还有一些文件的元信息，如权限，修改时间，大小等。**用`stat <filename>`命令可以查看**。
* 对于目录，天生具有两个硬链接，一个在其父目录下，一个是本身下面的`.`，所以**一个目录的links = 2 + “普通”子目录个数**（“普通”的意思是不包含`.`和`..`）
* 考虑到查找文件的性能，linux源码中规定**一个目录下最多只能包含3200个子文件（考虑`.`和`..`还要减去2）**，所以目录文件的大小并不能达到单个文件的理论最大值[（来源）](http://www.51testing.com/html/38/225738-236959.html)
* 一图胜千言。下图中的黄色表格是目录的data block

{% asset_img inode.png 文件查找 %}

### ubuntu 18.04 安装opencv
参考这个[链接](https://linuxconfig.org/install-opencv-on-ubuntu-18-04-bionic-beaver-linux)
cntk依赖3.1版，这个目前会安装3.2版，一个workaround是创建软链接欺骗cntk。

### `xx > /dev/null 2>&1` 的意义
* 意义：将xx的STDOUT重定位到/dev/null(黑洞), STDERR重定位到STDOUT，总的来说就是把程序的任何输出都丢到黑洞。
* 1前面为什么要加&：如果不加，STDERR就会重定位到一个文件，文件名为'1'。加&予以区分。[(来源)](https://www.xaprb.com/blog/2006/06/06/what-does-devnull-21-mean/)
---------------------------------
## C++
### 将C++ template的声明和实现分开， 编译时链接错误
template的声明和实现必须放在同一个文件中。参考下面的两个链接。  
1. [StackOverflow](https://stackoverflow.com/questions/1353973/c-template-linking-error)  
2. [知乎](https://www.zhihu.com/question/31845821)

-----------------------------------
## Shell

### shell tutorial
[中文](http://www.runoob.com/linux/linux-shell.html)

[英文](https://www.tutorialspoint.com/unix/)

### Syntax error: Bad for loop variable
检查你是不是用sh执行脚本的？是的话换bash看看。sh不支持for循环。

### bash脚本比较浮点数大小
bash本身只支持比较整数，要比较浮点数大小用下面的方式[(来源)](https://stackoverflow.com/questions/9939546/comparison-of-integer-and-floating-point-numbers-in-shell-script):
```bash
if [ $(echo "23.3 > 7.3" | bc) -ne 0 ] 
then 
  echo "wassup"
fi
```
## bash脚本中的布尔表达式
需要注意的是，bash脚本中的布尔表达式不会像C语言中一样被优化。这可能是引起下面错误的原因。

```
[: too many arguments
[: unary operator expected
```

---------------------------------
## git
官方文档是坠吼的：[English](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control)-[中文](https://git-scm.com/book/zh/v2)
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


## MXNet
### 多机训练
参考以下两个官方文档：

[Large Scale Image Classification](https://mxnet.incubator.apache.org/tutorials/vision/large_scale_classification.html)

[Distributed Training in MXNet](https://mxnet.incubator.apache.org/faq/distributed_training.html)

### Trouble shooting
多机训练时可能报错：

```
mxnet/src/io/iter_image_recordio_2.cc:318: Check failed: !overflow number of input images must be bigger than the batch size
```
出现这个问题的原因是验证集中\#example < \#worker * batch_size。解决方案：

1. 用更小的batch_size
2. 减少worker数量
3. 在训练时禁用验证集
