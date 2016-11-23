title: Open vSwitch安装与使用
toc: true
date: 2016-11-23 14:26:26
tags: 
- openvswitch
- openflow
categories: openvswitch
---

Open vSwitch是Apache 2.0协议下，实现分布式虚拟多层网络交换机功能的产品级开源软件，其目的是为硬件虚拟化环境提供交换机堆栈，支持计算机网络中使用的多种协议和标准

<!--more-->
**Title: [Open vSwitch安装与使用](https://aidaizyy.github.io/openvswitch-build)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2016-11-23](http://aidaizyy.github.io)**

本文使用的服务器操作系统发行版本为CentOS 6.3，kernel版本为2.6.32-279.el6.x86_64，Open vSwitch版本为2.4.1。

## 下载源码

下载Open vSwitch，目前最新的版本为2.6.1（发布于2016.11.2），只支持kernel 3.10-4.7。因为服务器的kernel版本（2.6.32-279）较低，经测试2.4.1可以正常使用（支持kernel 2.6.32-4.0），更高的还有2.5.1(支持kernel 2.6.32-4.3)。
``` bash
sudo wget http://openvswitch.org/releases/openvswitch-2.4.1.tar.gz
```
如需其他版本，把上面的2.4.1替换为2.6.1或者2.5.1或者更低的版本。
或者直接到官网[](http://openvswitch.org/download)下载。

## 准备环境

安装Open vSwitch需要准备的环境，可以参考[](https://github.com/openvswitch/ovs/blob/master/INSTALL.rst)的"Build Requirements"部分。
摘自官方文档：
需要的软件：
- GNU make
- 编译器：GCC 4.x/Clang 3.4
- libssl：可选，在需要连接ovs（Open vSwitch）到OpenFlows控制器时推荐安装
- libcap-ng：可选，在需要非root用户使用root权限运行ovs后台程序时推荐安装
- Pyhton 2.7
其他情况的软件及需要的内核模块自行参考官方文档。

如果要编译内核模块，需要与内核版本一致的内核源代码，通常位于`/usr/src/kernels/<version>`或者`/usr/src/<versio>`。
``` bash
cd /lib/modules/<version>
ls -l build
```
<version>指内核版本，用`uname -r`得到。
如果打印了一个目录列表，直接进行编译；
如果打印了`No such file or directory error`，执行以下操作：
``` bash
cd /lib/modules/<version>
rm build
ln -s /usr/src/kernels/<version> build
```
或者为
``` bash
cd /lib/modules/<version>
rm build
ln -s /usr/src/<version> build
```
重复上面的步骤验证是否生效。

## 编译安装

``` bash
tar -xvzf openvswitch-2.4.1.tar.gz
cd openvswitch-2.4.1
./configure --with-linux=/lib/modules/$(uname -r)/build
```
配置编译内核模块，如果不需要基于内核的交换机（可以只运行在用户态空间中），即直接`./configure`。
其他配置，参考上面链接的官方文档。

``` bash
make
make install
make modules_install
```
`make modules_install`编译内核模块，可以不用执行。
在编译内核模块的过程中，需要`/lib/modules/<version>/build/include/generated/utsrelease.h`，可能会遇到较低kenrel版本中的指定位置并没有文件`utsrelease.h`的错误，我们可以在`<version>/build/include/linux/utsrelease.h`找到该文件，把它复制到`.../generated/utsrelease.h`，重新编译。

## 加载模块

如果不需要基于内核的交换机，没有编译内核模块，可以跳过这一步。
``` bash
modprobe openvswitch
```

因为openvswitch模块与linux的bridge模块冲突，所以如果发生冲突，不能加载ovs的内核模块时，先卸载掉bridge模块。
``` bash
rmmod bridge
modprobe openvswitch
```

用`lsmod`查看已加载的所有模块，验证是否加载ovs的内核模块成功。
``` bash
lsmod | grep openvswitch
```

如果一直加载模块不成功，可以用命令`modinfo openvswitch`查看该模块的信息，内核版本以及依赖关系等。或者用`dmesg | tail`查看kernel的日志信息。

## 初始操作

遇到权限不够时，使用`sudo`命令或`root`用户，或者`libcap-ng`。

``` bash
mkdir -p /usr/local/etc/openvswitch
cd openvswitch-2.4.1
ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema
```
创建配置数据库
`vswitch.ovsschema`是一个数据库模板，存放在`openvswitch-2.4.1/vswitchd/vswitch.ovsschema`。
`conf.db`是数据库文件，是`vswith.ovsschema`的一份拷贝。

``` bash
mkdir -p /usr/local/var/run/openvswitch
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
    --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
    --private-key=db:Open_vSwitch,SSL,private_key \
    --certificate=db:Open_vSwitch,SSL,certificate \
    --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
    --pidfile --detach
```
创建连接到上面创建的配置数据库的Unix domain socket，以便管理员能管理数据库。
如果不需要SSL支持，删除掉`--private`，`--certificate`，`--bootstrap`。
执行以上操作后，开启了进程`ovsdb-server`。

``` bash
ovs-vsctl --no-wait init
```
对数据库进行初始化。

``` bash
ovs-vswitchd --pidfile --detach --log-file
```
开启ovs后台程序，连接到上面创建的Unix domain socket。一个后台程序可以管理和控制本机上任意数量的ovs交换机。
`--pidfile`的意思是创建一个运行的进程文件，默认路径为`/usr/local/var/run/openvswitch/`，可以用`ovs-appctl`管理该后台程序。
`--detach`的意思是在后台运行。
`--log-file`的意思是创建一个日志文件，默认路径为`/usr/local/var/log/openvswitch/`，可以查看该后台程序的日志。
其他参数可通过`man ovs-vswitchd`查看。

## 应用举例

`ovs-vsctl`命令主要是把配置信息更新到数据库中。

### 添加网桥
创建ovs网桥br0
``` bash
ovs-vsctl add-br br0
```
ovs网桥就表示以太网交换机（Switch）。
如果没有加载openvswitch内核模块，以上操作会报错，在日志文件中可以得到详情。如果想要ovs完全运行在用户态空间中，不使用内核模块，进行以下操作：
``` bash
ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev
```
`--`替代`ovs-vsctl`命令，也可以拆成两个语句执行。
后面的操作表示把数据库中ovs网桥br0的datapath_type属性的值设为netdev，对br0的其他属性设置操作类似。
netdev表示用户态数据通路，system表示内核数据通路。

删除ovs网桥br0
``` bash
ovs-vsctl del-br br0
```

### 添加端口
为br0添加端口p0
``` bash
ovs-vsctl add-port br0 p0
```
同样会报错，因为根本实际没有p0这个端口。
我们把p0的类型设置为虚拟端口可以解决这个问题。
``` bash
ovs-vsctl set interface p0 type=internal
```
interface是连接到port的网络接口设备，一对一关系，可以直接理解为port。
其他类型还有system、tap、geneve、gre、ipsec_gre、vxlan、lisp、stt、patch、null等。

删除端口p0
``` bash
ovs-vsctl del-port p0
```

### 网桥接管以太网卡
br0接管以太网卡端口eth0
``` bash
ovs-vsctl add-port br0 eth0
```
因为eth0是实际存在的端口，不需要特意设置类型为internal。
这时存在一个问题，以上操作执行后，eth0直接断网，不能连接到外网，如果使用ssh连接的服务器的就要小心了。（实际使用中发现只有在内核模式下才会断网，用户态模式下不会断网。）
要解决这个问题，只需要把eth0的相同ip/子网掩码/网关等设置移植给br0即可。
比如eth0的ip为192.168.1.100，子网掩码为255.255.0.0，网关为192.168.0.254。
``` bash
ifconfig br0 192.168.1.100 netmask 255.255.0.0
ifconfig br0 up
ifconfig eth0 0.0.0.0
```
注意还要把eth0的ip清空后。
在清空eth0的ip之前，最好执行`route -n`命令，观察eth0现有的路由设置，避免br0的路由设置出错。
``` bash
route add default gw 192.168.0.254
```
上面设置了默认网关，其他的路由设置自行查询`route`命令用法设置。
这时能ping通外网的话，表示设置成功。

### 虚拟端口连接外网
实际上这时p0在网桥中，已经连接到外网了，但还不能使用，因为p0还没有up。
同样设置ip和子网掩码，然后用混杂模式up。
``` bash
ifconfig p0 192.168.1.101 netmask 255.255.0.0
ifconfig p0 promisc up
```
混杂模式可以接收非本ip的数据包，如果不使用混杂模式，接收到非本ip的数据包直接丢弃，在交换机该场景下需要使用混杂模式。

这时通过指定ping外网，就可以验证是否成功。
``` bash
ping -I p0 http://www.baidu.com
```

## 参考资料

* Open vSwitch官网：http://openvswitch.org
* Github地址：https://github.com/openvswitch/ovs
* http://www.ibm.com/developerworks/cn/cloud/library/1401_zhaoyi_openswitch/
