---
title: Install cntk on Ubuntu 18.04
date: 2018-08-17 15:42:16
tags: [linux]
---

按照官网教程[[1]](https://docs.microsoft.com/en-us/cognitive-toolkit/setup-linux-python?tabs=cntkpy251)，用pip安装cntk仅需简单两步，但是在实际操作中，安装完毕后可能会报错。本文对安装过程做简单总结。
<!--more-->
## 安装过程
顺次执行下列命令
```
sudo apt install openmpi-bin
```
(cntk用的是mpi而不是parameter server架构？？？)
```
pip install cntk
```
(如果要安装支持gpu的版本，请将cntk替换为cntk-gpu)

然后用下面的命令检查cntk是否安装成功
```
python -c "import cntk; print(cntk.__version__)"
```

## Troubleshooting
### 找不到libmpi_cxx.so.1或libmpi.so.12
运行 `python -c "import cntk; print(cntk.__version__)"`时可能显示以下错误
```
leizhu@pc-office:~$ python -c "import cntk; print(cntk.__version__)"
Traceback (most recent call last):
  File "/home/leizhu/anaconda3/lib/python3.6/site-packages/cntk/cntk_py.py", line 18, in swig_import_helper
    return importlib.import_module(mname)
  File "/home/leizhu/anaconda3/lib/python3.6/importlib/__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 994, in _gcd_import
  File "<frozen importlib._bootstrap>", line 971, in _find_and_load
  File "<frozen importlib._bootstrap>", line 953, in _find_and_load_unlocked
ModuleNotFoundError: No module named 'cntk._cntk_py'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/leizhu/anaconda3/lib/python3.6/site-packages/cntk/__init__.py", line 17, in <module>
    from . import cntk_py
  File "/home/leizhu/anaconda3/lib/python3.6/site-packages/cntk/cntk_py.py", line 21, in <module>
    _cntk_py = swig_import_helper()
  File "/home/leizhu/anaconda3/lib/python3.6/site-packages/cntk/cntk_py.py", line 20, in swig_import_helper
    return importlib.import_module('_cntk_py')
  File "/home/leizhu/anaconda3/lib/python3.6/importlib/__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
ImportError: libmpi_cxx.so.1: cannot open shared object file: No such file or directory
```
执行下面的命令解决[[2]](https://tweaks-tips.blogspot.com/2017/12/microsoft-cntk-libmpi-importerror.html)
```
sudo ln -s /usr/lib/x86_64-linux-gnu/libmpi_cxx.so.20 /usr/lib/x86_64-linux-gnu/libmpi_cxx.so.1
```
对于找不到libmpi.so.12的情况同上处理
```
sudo ln -s /usr/lib/x86_64-linux-gnu/libmpi.so.20.10.1 /usr/lib/x86_64-linux-gnu/libmpi.so.12
```
此时再执行`python -c "import cntk; print(cntk.__version__)"`应正常输出版本号。

### 找不到libpng12.so.0和libjasper.so.1
后来在试跑Alexnet的时候又遇到了RuntimeError，先是找不到libpng12.so.0

```
RuntimeError: Plugin not found: 'Cntk.Deserializers.Image-2.5.1.so' (error: libpng12.so.0: cannot open shared object file: No such file or directory)
```
解决方式如下[[3]](https://github.com/tcoopman/image-webpack-loader/issues/95)
```
wget -q -O /tmp/libpng12.deb http://mirrors.kernel.org/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1_amd64.deb
sudo dpkg -i /tmp/libpng12.deb
rm /tmp/libpng12.deb
```
然后又找不到libjasper.so.1
```
RuntimeError: Plugin not found: 'Cntk.Deserializers.Image-2.5.1.so' (error: libjasper.so.1: cannot open shared object file: No such file or directory)
```
解决方法[[4]](https://stackoverflow.com/questions/43484357/opencv-in-ubuntu-17-04/43507858)：
```
sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
sudo apt update
sudo apt install libjasper1 libjasper-dev
```


## 参考
[1]https://docs.microsoft.com/en-us/cognitive-toolkit/setup-linux-python?tabs=cntkpy251

[2]https://tweaks-tips.blogspot.com/2017/12/microsoft-cntk-libmpi-importerror.html

[3]https://github.com/tcoopman/image-webpack-loader/issues/95

[4]https://stackoverflow.com/questions/43484357/opencv-in-ubuntu-17-04/43507858
