---
title: cuda编程入门
tags:
  - cuda
category: 历史进程
date: 2018-09-27 22:06:51
---


最近在学习cuda，这里po出我使用的资料和遇到的问题。
<!--More-->
## cuda是什么

### CUDA的定义
CUDA的全称是Compute Unified Device Architecture，统一计算架构。以下内容摘自[官方博客](https://blogs.nvidia.com/blog/2012/09/10/what-is-cuda-2/)
>Most people confuse CUDA for a language or maybe an API. It is not.  
>It’s more than that. CUDA is a parallel computing platform and programming model that makes using a GPU for general purpose computing simple and elegant. The developer still programs in the familiar C, C++, Fortran, or an ever expanding list of supported languages, and incorporates extensions of these languages in the form of a few basic keywords.  

意思是CUDA不是一门语言（你仍然使用（扩展了的）C/C++, Fortran以及其语言编程），也不是API，而是一个并行编程平台和一种编程模型，它使得用GPU来做通用计算简单优雅。下面的这张图来自[CS344](https://www.youtube.com/watch?v=F620ommtjqk&list=PLAwxTw4SYaPnFKojVQrmyOGFCqHTxfdv2)课程，我觉得比较形象地解释了CUDA的编程模型：  
{% asset_img cuda.png cuda %}

### cuda与cuDNN的关系
cudnn全称是CUDA Deep Neural Network library，顾名思义，这是一个**基于cuda**的为深度神经网络优化的**库**。所以cuda比cuDNN更基础，更底层。

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

## Problem shooting
### 为什么cudaMalloc中第一个参数是void**类型，而不是单指针？
提示：传值？传引用？
[这个链接](https://devtalk.nvidia.com/default/topic/418460/-void-amp-d-why-two-/)中_teju_给出了很好的回答(5楼)。

### cmake编译cuda?
参考[这里](https://www.cnblogs.com/haiyang21/p/7788063.html).

### 编译时遇到错误:undefined reference to `cv::imread(cv::String const&, int)'
跟安装的opencv版本有关, opencv3的接口有所改变，opencv3中将imread的实现放到了libopencv_imgcodecs模块中，因此编译时应该加上`-lopencv_imgcodecs`。
参看stackoverflow上的[这个问题](https://stackoverflow.com/questions/34497099/opencv-undefined-reference-to-imread).


顺便一提一个找解决方法时发现的一个很好用的工具[pkg-config](https://zh.wikipedia.org/wiki/Pkg-config), 使用示例：

```bash
leizhu@pc-office:~/git_repo/cs344/Problem Sets/Problem Set 1$ pkg-config --libs opencv
-lopencv_shape -lopencv_stitching -lopencv_superres -lopencv_videostab -lopencv_aruco -lopencv_bgsegm -lopencv_bioinspired -lopencv_ccalib -lopencv_datasets -lopencv_dpm -lopencv_face -lopencv_freetype -lopencv_fuzzy -lopencv_hdf -lopencv_line_descriptor -lopencv_optflow -lopencv_video -lopencv_plot -lopencv_reg -lopencv_saliency -lopencv_stereo -lopencv_structured_light -lopencv_phase_unwrapping -lopencv_rgbd -lopencv_viz -lopencv_surface_matching -lopencv_text -lopencv_ximgproc -lopencv_calib3d -lopencv_features2d -lopencv_flann -lopencv_xobjdetect -lopencv_objdetect -lopencv_ml -lopencv_xphoto -lopencv_highgui -lopencv_videoio -lopencv_imgcodecs -lopencv_photo -lopencv_imgproc -lopencv_core
leizhu@pc-office:~/git_repo/cs344/Problem Sets/Problem Set 1$ pkg-config --cflags opencv
-I/usr/include/opencv
leizhu@pc-office:~/git_repo/cs344/Problem Sets/Problem Set 1$ pkg-config --modversion opencv
3.2.0
```
