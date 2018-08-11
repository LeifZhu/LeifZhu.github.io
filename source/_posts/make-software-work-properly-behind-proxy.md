---
title: Make software work properly behind proxy
date: 2018-08-10 12:00:58
tags: [linux, 踩坑记录]
---

由于系里的以太网访问互联网必须经过代理。然而我发现如果仅仅设置系统代理，一些命令(apt update, npm等)仍然无法建立连接，而需要单独设置。记录踩坑过程如下。我使用的系统是Ubuntu 18.04 LTS.

# 系统代理
有一些软件（比如Chrome和Firefox, 命令行中的pip）会默认使用系统代理，也就是说，只要你正确设置了系统代理，这部分软件就能正常联网了。

## 通过图形界面设置
前往Settings -> Network -> Network Proxy -> Manual， 然后如下图所示

![](/images/system_proxy.png)

**注意**：填写的地址不要包含“http://”或"https://"字段,直接填写域名或ip.
<font size=1>(一定要皮一下加上？那好，在命令行下执行"echo $HTTP_PROXY"，看到了什么？在我测试的Ubuntu 18.04下此时仍然可以正常上网，但是一些命令行软件却无法建立互联网连接了) </font>

## 通过命令行设置
当然，你也可以在命令行下修改，也就是HTTPS_PROXY和HTTP_PROXY加入环境变量。但是由于在图形界面添加的代理没有问题，我没有尝试。有点担心这样能否对对图形界面的程序（比如浏览器）生效。

**注意**：不管是HTTPS_PROXY还是HTTP_PROXY,都应该设置为"http://..."，如果你将HTTPS_PROXY设为"https://..."，有些软件就不能正常使用了，请参考[[3]](https://serverfault.com/questions/817680/https-through-an-http-only-proxy)。

## 关于GUI和命令行设置代理的区别
一个非常有趣的现象是，当我通过GUI设置代理之后，尽管在终端中能通过
```
echo $HTTP_PROXY
echo $HTTPS_PROXY
```
查看代理环境变量，但是却无法在/etc/environment或者/etc/bash.bashrc或者/etc/.bashrc中发现关于这两个环境变量的记录。我原以为通过GUI设置就会将代理变量写入这些文件。
[[1]](https://askubuntu.com/questions/830796/proxy-config-gui-vs-setting-http-proxy-in-etc-environment)中提到：
1. 如果在GUI设置时apply systme wide (我的Ubuntu 18.04没有该选项)，那么环境变量就会写入/etc/environment 和 /etc/apt/apt.conf， 否则代理变量只存放在gsettings database(就是GUI的数据库？)中。
2. 通过在.bashrc添加代理环境变量通过不会对图形界面程序生效。

# 不默认使用系统代理的软件
## apt
设置完系统代理后，sudo apt install命令可以正常联网安装软件了，奇怪的是sudo apt update 和 sudo apt add-apt-repository均无法建立互联网连接。一个快速的解决方式如下
```
sudo -E apt install sofware-name
sudo -E apt add-apt-repository ppa:xx/yy
```
关于这么做的原理，可以参考[[2]](https://stackoverflow.com/questions/8633461/how-to-keep-environment-variables-when-using-sudo)，简单来说，sudo执行命令时默认不会保留当前用户的环境变量，而-E参数使得环境变量得以保留。（那么为啥不加-E参数时, sudo apt install没毛病呢? emmmmm, 我也不知道）

当然也可以在apt内设置代理:
```
sudo vim /etc/apt/apt.conf

然后在apt.conf中添加下面两行并保存
Acquire::http::proxy "http://lgn:pwd@proxy_sever:proxy_port";
Acquire::https::proxy "http://lgn:pwd@proxy_sever:proxy_port";
如果代理服务器不需要账户和密码则不需要lgn:pwd@字段。
```

## git
系统代理设置完成后使用git克隆远程github仓库发现没有响应，使用的命令如下：
```
git clone git@github.com:LeifZhu/LeifZhu.github.io.git
```
因为这时使用的是ssh协议，穿不透代理。你可以选择换成clone with https，即执行
```
git clone https://github.com/LeifZhu/LeifZhu.github.io.git
```
或者参考[[4]](https://unix.stackexchange.com/questions/190490/how-to-use-ssh-over-http-or-https)或者[[5]](https://blog.csdn.net/twilightdream/article/details/78260394)通过https支持ssh协议。以我的使用为例
先安装corkscrew
```bash
sudo apt install corkscrew
```
然后修改~/.ssh/config,添加
```bash
Host github.com
	User git
	Hostname ssh.github.com
	Port 443
	ProxyCommand /usr/bin/corkscrew proxy.cse.cuhk.edu.hk 8000 %h %p
	IdentityFile ~/.ssh/id_rsa
```

# 总结
1. 在图形界面设置系统代理时不要加上协议头"http://"
2. 即使设置了系统代理，一些软件也可能需要在软件内再设置代理。如果你发现浏览器能上网而命令行软件不能，那么你应该考虑配置软件内代理，或者对于sudo命令加上-E参数。
3. 无论是HTTP_PROXY,还是HTTPS_PROXY,都应该设置为"http://..."


# 参考
[1]https://askubuntu.com/questions/830796/proxy-config-gui-vs-setting-http-proxy-in-etc-environment

[2]https://stackoverflow.com/questions/8633461/how-to-keep-environment-variables-when-using-sudo

[3]
https://serverfault.com/questions/817680/https-through-an-http-only-proxy

[4]https://unix.stackexchange.com/questions/190490/how-to-use-ssh-over-http-or-https


[5]https://blog.csdn.net/twilightdream/article/details/78260394