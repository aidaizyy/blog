title: SSH秘钥与SSHFS挂载
toc: true
date: 2016-08-25 15:37:54
tags:
categories: linux
---

SSH通过秘钥（公钥与私钥）完成免密SSH连接。
SSHFS挂载远程目录到本地，本地操作同步到远程目录。
<!--more-->
**Title: [SSH秘钥与SSHFS挂载](https://aidaizyy.github.io/ssh)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2016-08-25](http://aidaizyy.github.io)**

## 查看已有的秘钥
``` bash
cd ~/.ssh
ls
```
以.pub结尾的文件就是公钥，而与.pub前字符串相同的文件就是与之对应的私钥。
## 创建新的秘钥
``` bash
ssh-keygen -t rsa -C “usr@email”
```
参数“rsa”是加密方式，其他可选的加密方式有dsa；
参数“-C”后面接注释。
创建过程中会要求输入存放的文件名，直接回车默认是id_rsa和id_rsa.pub或id_dsa和id_dsa.pub；
还会要求输入密码，直接回车默认为空。
## 机器A登录机器B
在.ssh目录下有authorized_keys文件，把机器A生成的公钥拷贝到机器B的authorized_keys文件中，机器A就可以免密登录机器B；
## 设置文件和目录权限
一般依据默认权限，如果不小心删除了，新建文件和目录，就必须设置权限。
``` bash
chmod 600 authorized_keys
chmod 700 -R .ssh
```
## 添加秘钥到SSH-AGENT
ssh-agent就是秘钥管理器，需要把私钥添加进去才可以使用ssh;
先确保ssh-agent是否可用：
``` bash
eval "$(ssh-agent -s)"
-> Agent pid 59566
```
然后添加私钥：
``` bash
ssh-add ~/.ssh/id_rsa
```
可以查看ssh-agent中已有的私钥：
``` bash
ssh-add -l
```
## SSH连接服务器
``` bash
ssh usrname@ipaddr
```
logout：注销用户，exit：逐层退出控制台。
## GitHub和Coding公钥添加
在各自的设置页面，把公钥，也就是.pub文件中的内容拷贝到设置中指定的输入框即可生效；对应的私钥需要添加到ssh-agent中。
github的ssh秘钥是否生效可以进行测试：
``` bash
ssh git@github.com
```
## SSHFS挂载
在配置ssh秘钥免密登录后，可以用sshfs把远程目录挂载到本地上，比如把服务器上的目录挂载过来，通过本地的编程环境（IDE、插件等）编辑代码文件。
Linux系统直接通过apt-get或者yum安装sshfs。
Mac OS X系统需要安装osxfuse和其对应的sshfs：
安装FUSE for OS X：https://osxfuse.github.io；
安装SSHFS：https://github.com/osxfuse/sshfs/releases；
两者都可以通过brew安装。
``` bash
sshfs -o allow_other user@host:dir localdir
```
user指远程用户名，host指远程主机地址，dir指要挂载的远程目录，localdir指挂载到的本地目录。
-o 后面接相关参数，比如：
* -o reconnect：自动重连
* -o allow_other：无视用户权限
* -o cache=yes：支持cache

卸载远程目录，使用卸载设备命令“umount”。
``` bash
umount localdir
```

其他相关参数可以查阅资料或者通过`sshfs -h`了解。

** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" /> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>
