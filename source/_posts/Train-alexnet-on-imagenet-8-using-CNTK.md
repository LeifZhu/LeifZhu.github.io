---
title: Train Alexnet on imagenet-8 using CNTK
date: 2018-08-19 21:51:25
tags: [deep learning]
---

为了做某个与项目有关的评测，我需要在imagenet-8（imagenet的一个子集，只包含前八类的训练数据和测试数据）上用CNTK训练Alexnet。官方提供了Alexnet基于CNTK的python实现，但是却无法直接执行，因为脚本需要数据集中包含一个叫map file的东西。
<!--more-->

## 问题
在开始训练前，我已经准备好了imagenet8训练集放在~/dataset/目录下,目录结构如下
```
leizhu@pc-office:~/dataset$ tree -d
.
└── imagenet8
    ├── mxnet-format
    │   ├── train
    │   ├── train_meta
    │   ├── val
    │   └── val_meta
    └── raw
        ├── train
        │   ├── n01440764
        │   ├── n01443537
        │   ├── n01484850
        │   ├── n01491361
        │   ├── n01494475
        │   ├── n01496331
        │   ├── n01498041
        │   └── n01514668
        └── validation
            ├── n01440764
            ├── n01443537
            ├── n01484850
            ├── n01491361
            ├── n01494475
            ├── n01496331
            ├── n01498041
            └── n01514668

25 directories

```
然后我从CNTK的github仓库中下载了Alexnet的Python实现[[1]](https://github.com/Microsoft/CNTK/blob/master/Examples/Image/Classification/AlexNet/Python/AlexNet_ImageNet_Distributed.py)，并执行该脚本：
```
python AlexNet_ImageNet_Distributed.py --datadir ~/dataset/imagenet8/raw/
```
结果报错
```
RuntimeError: File '/home/leizhu/dataset/imagenet8/raw/train_map.txt' does not exist.
```
那么train_map.txt是什么呢？几经周折找到一个实例文件[[2]](https://github.com/Microsoft/CNTK/blob/master/Tests/EndToEndTests/Image/AlexNet/train_map.txt)，它长这个样子：
```
val1024.zip@/ILSVRC2012_val_00000001.JPEG	65
val1024.zip@/ILSVRC2012_val_00000002.JPEG	970
val1024.zip@/ILSVRC2012_val_00000003.JPEG	230
val1024.zip@/ILSVRC2012_val_00000004.JPEG	809
val1024.zip@/ILSVRC2012_val_00000005.JPEG	516
val1024.zip@/ILSVRC2012_val_00000006.JPEG	57
val1024.zip@/ILSVRC2012_val_00000007.JPEG	334
val1024.zip@/ILSVRC2012_val_00000008.JPEG	415

```
看起来每一行都是<图象路径> + <标签>。根据链接[[3]](https://cntk.ai/pythondocs/CNTK_201A_CIFAR-10_DataLoader.html)中的代码，应该就是了。

## 解决方案
编写下面的python脚本
```python
import os
import argparse


def generate_map_file(abs_data_dir):
    train_dir = os.path.join(abs_data_dir, "train")
    val_dir = os.path.join(abs_data_dir, "validation")
    class_dirs = os.listdir(train_dir)
    
    with open(os.path.join(abs_data_dir, "train_map.txt"), "w") as f:
        for i in range(len(class_dirs)):
            pic_dirname = os.path.join(train_dir, class_dirs[i])
            for pic in os.scandir(pic_dirname):
                if pic.is_file() and pic.name.endswith('.JPEG'):
                    f.write("%s\t%d\n"%(os.path.join(pic_dirname,pic),i))
                    
    with open(os.path.join(abs_data_dir, "val_map.txt"), "w") as f:
        for i in range(len(class_dirs)):
            pic_dirname = os.path.join(val_dir, class_dirs[i])
            for pic in os.scandir(pic_dirname):
                 if pic.is_file() and pic.name.endswith('.JPEG'):
                    f.write("%s\t%d\n"%(os.path.join(pic_dirname,pic),i))
    
    print("generate mapfile successfully!\n")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description = "generate mapfile")
    parser.add_argument("--data_dir", type=str, default="/home/leizhu/dataset/imagenet8/raw/",
                    help="absolute path of data directory")
    args = parser.parse_args()
    generate_map_file(args.data_dir)
```
然后执行
```
python map_file.py --data_dir ~/dataset/imagenet8/raw
```
再进行训练就没有问题了。

如果你要根据元文件(应该是个xml?我没有原始数据集，太大了下不动)为原始的imagenet  ILSVRC数据集生成map file，请参考[这里[4]](https://github.com/Microsoft/CNTK/issues/2091)。

## referrence
[1]https://github.com/Microsoft/CNTK/blob/master/Examples/Image/Classification/AlexNet/Python/AlexNet_ImageNet_Distributed.py

[2]https://github.com/Microsoft/CNTK/blob/master/Tests/EndToEndTests/Image/AlexNet/train_map.txt

[3]https://cntk.ai/pythondocs/CNTK_201A_CIFAR-10_DataLoader.html

[4]https://github.com/Microsoft/CNTK/issues/2091