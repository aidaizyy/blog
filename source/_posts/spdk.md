title: SPDK简介
date: 2016-08-15 23:02:27
tags:
- spdk
categories: spdk
toc: true
---

SPDK（Storage Performance Development Kit）是Intel发布的存储性能开发工具集。
> 原文：[《Introduction to the Storage Performance Development Kit (SPDK)》](https://software.intel.com/en-us/articles/introduction-to-the-storage-performance-development-kit-spdk)

<!--more-->
**Title: [SPDK简介](https://aidaizyy.github.io/spdk)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2016-08-16](http://aidaizyy.github.io)**


## 简介
固态存储媒介正在取代数据中心。目前这一代的闪存存储，比起传统的磁盘介质，在性能（performance）、功耗（power consumption）和机架密度（rack density）上具有显著的优势。这些优势将会继续增大，使闪存存储作为下一代媒介进入市场。

用户使用现在的固态媒介，比如Intel P3700非易失存储器标准（Non-Volatile Memory Express, NVMe）驱动，面临一个主要的挑战：因为吞吐量和延迟性能比传统的磁盘好太多，现在总的处理时间中，存储软件占用了更大的比例。换句话说，存储软件栈的性能和效率在整个存储系统中越来越重要。随着存储媒介继续发展，它将面临远远超过正在使用的软件体系结构的风险（即存储媒介受制于相关软件的不足而不能发挥全部性能），在接下来的几年中，存储媒介将会继续发展到一个令人难以置信的地步。

为了帮助存储OEM（设备代工厂）和ISV（独立软件开发商）整合硬件，Inte构造了一系列驱动和一个完善的，端对端引用存储体系结构，称为存储性能开发工具集（Storage Performance Development Kit, SPDK）。SPDK的目标是通过同时使用Intel的网络，处理器和存储技术来提高突出显著的效率和性能。通过运行为硬件设计的软件，SPDK已经证明很容易达到每秒钟数百万次I/O读取，通过使用许多处理器核心和许多NVMe驱动去存储，而不需要额外卸载硬件。Intel提供了其全部的Linux参考架构的源代码，在Intel的许可下无需付费，把用户态NVMe驱动通过01.org开源，开发工具集的其他内容将会在2016年开源。

## 软件体系结构概览
SPDK如何工作？达到这样的超高性能运用了两个关键技术：运行于用户态和轮询模式。让我们进一步了解这两个软件工程术语。

首先，我们的设备驱动代码运行在用户态意味着，在定义上，驱动代码不会运行在内核中。避免内核上下文切换和中断将会节省大量的处理开销，允许更多的时钟周期被用来做实际的数据存储。无视复杂的存储算法（去冗，加密，压缩，空白块存储），更少浪费的时间周期意味着更好的性能。

其次，轮询模式驱动（Polled Mode Drivers, PMDs）会持续工作，而不是被派遣工作。考虑在一个忙碌的周六晚上的市区打车，不断招手，一辆又一辆后座已经有乘客的计程车驶过。这样的等待是不可预测的，不可能说清楚花掉多少分钟之后能打到车。打车就像在传统的中断-派遣存储I/O驱动中等待一个包或一个数据块传输。另一方面，想象在机场打车的过程中，队首的计程车司机观察，只需要固定的几秒钟就可以载上乘客并驶向目的地。这就是PMDs怎么工作的，SPDK的所有组件怎么设计的。数据包和数据块被立即分派，因为等待花掉的时间变小，使得延迟更低，一致性延迟更多（抖动更少），吞吐量也得到提高。

SPDK由数个子组件构成，相互连接并共享用户态和轮询模式的共有部分。当构造端对端SPDK体系结构时，每个组件被构造用于克服遭遇到的特定的性能瓶颈。然而，每个组件也可以被整合进非SPDK体系结构，允许用户使用SPDK影响经验和技术而加速他们自己的软件。举例来说，用户态网络服务（Userspace Network Services, UNS）库被构造用来克服Linux内核TCP/IP协议栈的性能限制。通过构造一个用户态，轮序模式的TCP/IP协议栈，SPDK可以通过更少的处理器时钟周期处理TCP/IP包排序和处理，获得地更高的IOPS性能。

![SPDK Architecture](https://software.intel.com/sites/default/files/managed/a8/ff/introduction-to-the-storage-performance-development-kit-spdk-fig2.png)

一共三种基本类型的子组件：网络前端，处理框架，后端。

前端是由数据平面开发工具集（Data Plane Development Kit, DPDK）网卡驱动和UNS组成。DPDK提供一个高性能的在网卡中处理数据包的框架，提供一个从网卡到用户空间的数据到达的快速路径。然后UNS代码进行接管，破解TCP/IP数据包形成iSCSI命令。

这时，处理框架得到数据包内容并翻译iSCSI命令为SCSI块级命令。然而，在它把这些命令发送到后端驱动之前，SPDK提供一个API框架增加用户特定的功能——“special sauce”，到SPDK框架中（上图绿色框内）。很多例子，包括缓存，数据的去冗和压缩，磁盘阵列（RAID），和纠删码计算。这些功能都包含在SPDK中，虽然这些功能只是为了帮助我们构建真实世界的用例，不应该和产品级实现混淆。

最终，数据到达后端驱动，在这里与物理块设备交互发生，就是读或写。SPDK包括针对几种存储媒介的用户态PMDs；NVMe设备；Linux AIO设备，比如传统磁盘；基于块地址的内存应用的内存驱动（比如，RAMDISKS）；和可以用Intel I/O加速技术（Intel I/O Acceleration Technology，代号为Crystal Beach DMA，CBDMA）的设备。这套后端驱动涵盖了不同层次性能的存储设备，保证了几乎每种存储应用的相关性。

SPDK不适应所有的存储体系结构。这里有一些问题可能会帮助用户决定SPDK组件是否适合他们的体系结构。

**这个存储系统是否基于Linux？**
SPDK现在只在Linux上测试和支持。

**这个存储系统的高性能路径是否运行在用户态？**
SPDK通过仅在用户态下运行从网卡到磁盘的高性能路径，提高性能和效率。

**系统体系结构可以合并无锁的PMDs到它的线程模型吗？**
因为PMD持续运行在它们的线程中（而不是睡眠或者不用时让出处理器），所以它们有特殊的线程模型需求

**系统现在是否用DPDK处理网络数据包的工作负载**
DPDK包含了SPDK的框架，所以现在使用DPDK的用户就会发现与SPDK紧密整合非常有用。

**你们的许可模型可以使用非可再分发源吗？**
SPDK的一部分是作为开源可获得的，BSD许可组件（比如NVMe和CBDMA用户态驱动）。而其他部分暂时是Intel许可下（UNS和用户态iSCSI对象），但是它肯定会改变，所有的SPDK源代码免费提供。

**开发团队自己是否具有理解和解决问题的专业技能？**
Intel没有为相关软件提供支持的义务。当Intel和围绕SPDK的开源社区将付出商业上合理的努力去调出未修改的发布版本软件的潜在错误，任何情况下Intel都没有任务义务为用户提供针对该软件任何形式的维护和支持。

** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" /> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>
