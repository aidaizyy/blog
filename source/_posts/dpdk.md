title: DPDK编译运行
toc: true
date: 2015-04-28 14:46:52
tags: 
- dpdk
categories: dpdk
---

DPDK（Data Plane Development kit）是Intel发布的数据包处理转发套件。

<!--more-->
**Title: [dpdk编译运行](https://aidaizyy.github.io/dpdk)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2016-08-23](http://aidaizyy.github.io)**

## 下载源码

下载DPDK-2.0.0
``` bash
wget http://www.dpdk.org/browse/dpdk/snapshot/dpdk-2.0.0.tar.gz
```
或者直接访问http://www.dpdk.org/download/ 下载最新的版本。

解开压缩包
``` bash
tar -xvzf dpdk-2.0.0.tar.gz
```

## 准备环境

### linux kernel header

确保系统是否已安装linux kernel header，未安装则：
``` bash
sudo apt-get install linux-header-3.13.0-49-generic
```

linux kernel版本号由系统本身决定，以下命令查看：
``` bash
uname -r
```

kernel版本号必须大于2.6.33。
同时glibc版本号大于2.7。

### libpcap函数库

``` bash
sudo apt-get install libpcap-dev
```

### hugepages

查看kernel是否支持hugepapse
``` bash
grep -i huge /boot/config-3.13.0-49-generic
```
同样，kernel版本号由系统本身决定。
如果出现
``` bash
CONFIG_HUGETLBFS=y
CONFIG_HUGETLB_PAGE=y
```
则表示支持hugepages。

查看当前系统hugepages信息
``` bash
grep -i huge /proc/meminfo
```

配置hugepages
``` bash
vi /etc/sysctl.conf
#在文件底部添加
vm.nr_hugepages=512
#表示hugepages的页面数量

vi /etc/fstab
#在文件底部添加
huge /mnt/huge hugetlbfs defaults 0 0

mkdir /mnt/huge
chmod 777 /mnt/huge
```

重新启动后查看/proc/meminfo 就会发现hugepages已经加载。
``` bash
AnonHugePages:     53248 kB
HugePages_Total:     512
HugePages_Free:      512
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

## 编译安装
``` bash
cd dpdk-2.0.0
make install T=x86_64-native-linuxapp-gcc
```
x86_64指x86构架64位系统。如果是32位系统，将x86_64替换为i686。

官网给出的编译平台规范是ARCH-MACHINE-EXECENV-TOOLCHAIN
ARCH can be: i686, x86_64, ppc_64
MACHINE can be: native, ivshmem, power8
EXECENV can be: linuxapp, bsdapp
TOOLCHAIN can be: gcc, icc

这里在Ubuntu Linux 64位系统本地环境下gcc工具编译

也可以先设置，再编译。
``` bash
make config T=x86_64-native-linuxapp-gcc
make
```

make install会将编译后的文件放入新建的x86_64-native-linuxapp-gcc目录。
make config + make会将编译后的文件放入新建的build目录。

## 加载模块

``` bash
sudo modprobe uio
sudo insmod kmod/igb_uio.ko
```
uio是kernel自带的用户空间IO模块
igb_uio是dpdk编译的模块，出现在dpdk-2.0.0/build/kmod 或者dpdk-2.0.0/x86_64-native-linuxapp-gcc/kmod 目录中。
（在新版本中可直接用`sudo modprobe uio_pci_generic`替代`uio`和`igb_uio`）

## 绑定网卡

查看当前网卡信息
（在新版本中用`dpdk-devbind.py`替代`dpdk_nic_bind.py`）
``` bash
cd dpdk-2.0.0
./tools/dpdk_nic_bind.py --status

Network devices using kernel driver
===================================
0000:00:05.0 '82545EM Gigabit Ethernet Controller (Copper)' if=eth0 drv=e1000 unused= *Active*
0000:00:06.0 '82545EM Gigabit Ethernet Controller (Copper)' if=eth1 drv=e1000 unused= *Active*
0000:00:07.0 '82545EM Gigabit Ethernet Controller (Copper)' if=eth3 drv=e1000 unused= *Active*

Other network devices
=====================
<none>

```

绑定网卡
（在新版本中如果使用了`uio_pci_generic`，则把`--bind=`后的`igb_uio`换成`uio_pci_generic`）
``` bash
./tools/dpdk_nic_bind.py --bind=igb_uio 00:05.0
```

绑定之前，保证网卡处于非活跃状态
``` bash
ifconfig eth0 down
```

## 运行示例

运行helloworld示例
``` bash
#添加环境变量
export RTE_SDK=$SDK/dpdp-2.0.0
export RTE_TARGET=x86_64-native-linuxapp-gcc

#编译
cd /dpdk-2.0.0/example/helloworld
make

#运行
./build/helloworld -c 3 -n 2

hello from core 1
hello from core 0
```

这里的RTE_SDK指dpdk主目录的路径。

-c COREMASK -n NUM为必须参数
COREMASK: 一个十六进制位掩码表示分配的逻辑内核数量。
NUM: 一个十进制整数表示内存通道数量。

运行完成后，显示
hello from core 1
hello from core 0。

其他示例程序参数有不同要求，参见官方网站的说明文档。

## 脚本安装

DPDK提供了更简单的脚本安装。
在解开压缩包和设置好环境变量RTE_SDK和RTE_TARGET后，运行setup.sh脚本。
``` bash
cd /dpdk-2.0.0
./tools/setup.sh

----------------------------------------------------------
 Step 1: Select the DPDK environment to build
----------------------------------------------------------
[1] i686-native-linuxapp-gcc
[2] i686-native-linuxapp-icc
[3] ppc_64-power8-linuxapp-gcc
[4] x86_64-ivshmem-linuxapp-gcc
[5] x86_64-ivshmem-linuxapp-icc
[6] x86_64-native-bsdapp-clang
[7] x86_64-native-bsdapp-gcc
[8] x86_64-native-linuxapp-clang
[9] x86_64-native-linuxapp-gcc
[10] x86_64-native-linuxapp-icc
[11] x86_x32-native-linuxapp-gcc

----------------------------------------------------------
 Step 2: Setup linuxapp environment
----------------------------------------------------------
[12] Insert IGB UIO module
[13] Insert VFIO module
[14] Insert KNI module
[15] Setup hugepage mappings for non-NUMA systems
[16] Setup hugepage mappings for NUMA systems
[17] Display current Ethernet device settings
[18] Bind Ethernet device to IGB UIO module
[19] Bind Ethernet device to VFIO module
[20] Setup VFIO permissions

----------------------------------------------------------
 Step 3: Run test application for linuxapp environment
----------------------------------------------------------
[21] Run test application ($RTE_TARGET/app/test)
[22] Run testpmd application in interactive mode ($RTE_TARGET/app/testpmd)

----------------------------------------------------------
 Step 4: Other tools
----------------------------------------------------------
[23] List hugepage info from /proc/meminfo

----------------------------------------------------------
 Step 5: Uninstall and system cleanup
----------------------------------------------------------
[24] Uninstall all targets
[25] Unbind NICs from IGB UIO or VFIO driver
[26] Remove IGB UIO module
[27] Remove VFIO module
[28] Remove KNI module
[29] Remove hugepage mappings

[30] Exit Script

Option: 
```
按照脚本指示一步一步运行即可。
依次执行9-12-15-18就可以达到和上面一样的结果。
当然不同情况，脚本执行步骤不同。

## 示例程序

几个值得关注的示例程序。

- testpmd: 测试程序，可以在setup.sh脚本中运行或者在app/ 目录下。
	文档: http://www.dpdk.org/doc/guides/testpmd_app_ug/index.html

- l2fwd: 链路层转发程序，在example/ 目录下。
	example /目录下有很多其他值得关注的示例程序。
	文档: http://www.dpdk.org/doc/guides/sample_app_ug/index.html

- pktgen-dpdk: 基于DPDK的高速发包程序
	DPDK官方网站：http://www.dpdk.org/browse/apps/pktgen-dpdk 
	GitHub：http://github.com/pktgen/Pktgen-DPDK

** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" /> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>
