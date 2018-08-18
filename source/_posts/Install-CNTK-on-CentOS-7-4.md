---
title: Install CNTK on CentOS 7.4
date: 2018-08-19 00:56:23
tags:
---
 
 ```
[root@ssd02 alexnet-imagenet-8]# yum install openmpi-bin
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.01link.hk
 * epel: del-repos.extreme-ix.org
 * extras: centos.01link.hk
 * updates: centos.01link.hk
No package openmpi-bin available.
Error: Nothing to do
 ```
 
 ```
 pip install cntk
 ```
 
 ```
 [root@ssd02 alexnet-imagenet-8]# python -c "import cntk; print(cntk.__version__)"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/root/anaconda2/lib/python2.7/site-packages/cntk/__init__.py", line 17, in <module>
    from . import cntk_py
  File "/root/anaconda2/lib/python2.7/site-packages/cntk/cntk_py.py", line 21, in <module>
    _cntk_py = swig_import_helper()
  File "/root/anaconda2/lib/python2.7/site-packages/cntk/cntk_py.py", line 20, in swig_import_helper
    return importlib.import_module('_cntk_py')
  File "/root/anaconda2/lib/python2.7/importlib/__init__.py", line 37, in import_module
    __import__(name)
ImportError: libmpi_cxx.so.1: cannot open shared object file: No such file or directory
 ```
 
 ```
 sudo yum install openmpi
 ```
 
 ```
 [root@ssd02 alexnet-imagenet-8]# ls /usr/lib64 | grep mpi
openmpi
```

```
[root@ssd02 lib]# cp /usr/lib64/openmpi/lib/libmpi.so.12 /usr/lib64/libmpi.so.12
[root@ssd02 lib]# cp /usr/lib64/openmpi/lib/libmpi_cxx.so.1 /usr/lib64/libmpi_cxx.so.1
```

```
[root@ssd02 alexnet-imagenet-8]# python -c "import cntk; print(cntk.__version__)"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/root/anaconda2/lib/python2.7/site-packages/cntk/__init__.py", line 17, in <module>
    from . import cntk_py
  File "/root/anaconda2/lib/python2.7/site-packages/cntk/cntk_py.py", line 21, in <module>
    _cntk_py = swig_import_helper()
  File "/root/anaconda2/lib/python2.7/site-packages/cntk/cntk_py.py", line 20, in swig_import_helper
    return importlib.import_module('_cntk_py')
  File "/root/anaconda2/lib/python2.7/importlib/__init__.py", line 37, in import_module
    __import__(name)
ImportError: /lib64/libstdc++.so.6: version `CXXABI_1.3.8' not found (required by /root/anaconda2/lib/python2.7/site-packages/_cntk_py.so)
```
