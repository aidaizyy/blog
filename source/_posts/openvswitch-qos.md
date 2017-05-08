title: Open vSwitch的OpenFlow和QOS
toc: true
date: 2016-11-24 14:41:03
tags:
- openvswitch
- openflow
categories: openvswitch
---

OpenFlow协议是一种网络通信协议，属于数据链路层，可以控制几换几或者路由器的转发平面（forwarding plane)，借此改变网络数据包所走的网络路径。
Open vSwitch支持OpenFlow协议，就可以控制数据包的走向，还可以修改源目的地址，支持QOS（Quality of Service）等。
本文主要介绍Open vSwitch配置OpenFlow协议以及对QOS的支持。

<!--more-->
**Title: [Open vSwitch的OpenFlow和QOS](https://aidaizyy.github.io/openvswitch-qos)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2016-12-26](http://aidaizyy.github.io)**

## OpenFlow基本应用

这里有两篇参考文章，已经写得非常详细了：
- [《基于Open vSwitch的OpenFlow实践》](http://www.ibm.com/developerworks/cn/cloud/library/1401_zhaoyi_openswitch)
- [Open vSwitch Advanced Features](https://github.com/openvswitch/ovs/blob/master/Documentation/tutorials/ovs-advanced.rst)

交换机包括一个或多个流表，流表中的条目主要包括数据包要匹配的信息，匹配成功后要执行的操作和统计信息三部分。

``` bash
ovs-vsctl add-br br0
ovs-vsctl add-port br0 p0 -- set interface p0 type=internal
ovs-vsctl add-port br0 eth0
```
创建网桥br0，并添加虚拟端口p0和以太网端口eth0。
再对br0进行网络配置，使其能上外网，并将p0 up。（参考上一篇博文[《Open vSwitch安装与使用》](http://aidaiz.com/openvswitch-build)。）

`ovs-ofctl`是OpenFlow相关命令，详细参考`man ovs-ofctl`。
`ovs-ofctl show br0`命令是显示交换机br0的端口信息。
``` bash
$ ovs-ofctl show br0
FPT_FEATURES_REPLY (xid=0x2): dpid:00000025909765b0
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(eth0): addr:00:25:90:97:65:b0
     config:     0
     state:      0
     current:    1GB-FD COPPER AUTO_NEG
     advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG AUTO_PAUSE
     supported:  10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     speed: 1000 Mbps now, 1000 Mbps max
 2(p0): addr:d2:ca:93:d4:d5:0b
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br0): addr:00:25:90:97:65:b0
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```
dpid后面的字符串表示交换机br0的datapath id。
eht0前面的0和p0前面的1表示端口eth0和端口p0的OpenFlow端口的id。
其他的还有端口名称，端口状态等。

端口的OpenFlow id也可以修改，用`ovs-vsctl set interface p0 ofport_request=id`命令。
``` bash
$ ovs-vsctl set interface eth0 ofport_request=100
$ ovs-vsctl set interface p0 ofport_request=101
$ ovs-ofctl show br0
OFPT_FEATURES_REPLY (xid=0x2): dpid:00000025909765b0
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 100(eth0): addr:00:25:90:97:65:b0
     config:     0
     state:      0
     current:    1GB-FD COPPER AUTO_NEG
     advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG AUTO_PAUSE
     supported:  10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     speed: 1000 Mbps now, 1000 Mbps max
 101(p0): addr:fe:07:57:d1:46:f7
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br0): addr:00:25:90:97:65:b0
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```

``` bash
ovs-ofctl add-flow br0 priority,in_port=101,actions=normal
```
上面命令在br0上添加了一个流表条目，即一个OpenFlow规则，匹配内容是从端口id为101的端口出来的数据包，匹配成功后的操作为normal，即不做特殊处理。
prority表示优先级，prority值越高，优先级越高，其取值区间为0-65535，不显示指定的默认值为32768。
注意：OpenFlow规则语句中不能加空格；如果一定要加空格，整个规则语句必须要双引号括起来。比如：
``` bash
ovs-ofctl add-flow br0 "priority=10, in_port=101, actions=normal"
```

``` bash
$ ovs-ofctl add-flow br0 in_port=101,actions=normal
$ ovs-ofctl dump-flows br0
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=11.641s, table=0, n_packets=0, n_bytes=0, idle_age=11, priority,in_port=101 actions=NORMAL
 cookie=0x0, duration=61979.386s, table=0, n_packets=7766539, n_bytes=758342681, idle_age=0, priority=0 actions=NORMAL
```
`ovs-ofctl dump-flows br0`命令是查看交换机br0中所有的流表条目。
上图中出现了两条规则，第二条是我们创建br0时自动生成的，没有任何特殊操作，目的是为了统计数据包信息。
第一条规则，是我们创建的：
- duration: 该规则存在的时间
- table: 属于第0号流表
- n_packets: 匹配成功的数据包数量
- n_bytes: 匹配成功的数据包总大小
- idle_age: 与该规则相关的规则最近一次修改到现在的时间
我们设置的流表条目没有做任何处理，意义仅仅是统计数据包信息。
因为p0是虚拟端口，很少流过数据包，所以n_packets的值几乎不变。
``` bash
ping -I p0 www.baidu.com
```
指定p0端口去ping外网，一段时间后去执行`dump-flows`命令，可以看到n_packets的值剧烈增加。

除了`in_port`之外还有其他的匹配模式，除了`normal`之外还有其他的匹配成功的操作，具体可参加上面提到的文章[《基于Open vSwitch的OpenFlow实践》](http://www.ibm.com/developerworks/cn/cloud/library/1401_zhaoyi_openswitch)，还可以参考数据库[ovs-vswitchd.conf.db(5)](http://openvswitch.org/support/dist-docs/ovs-vswitchd.conf.db.5.html)。

其他相关命令还有：
``` bash
#查看交换机br0中所有的流表
ovs-ofctl dump-tables br0

#删除端口id为101的端口上所有的的流表条目
ovs-ofctl del-flows br0 in_port=101
```

关于OpenFlow的应用不再做更多介绍，可以参考上面列出的两篇文章。

## Open vSwitch的QOS

Open vSwitch关于QOS的官方资料，主要在：
- [ovs-vsctl(8)](http://openvswitch.org/support/dist-docs/ovs-vsctl.8.html)
- [ovs-vswitchd.conf.db(5)](http://openvswitch.org/support/dist-docs/ovs-vswitchd.conf.db.5.html)
- [Frequently Asked Questions: Quality of Service (QoS)](https://github.com/openvswitch/ovs/blob/master/Documentation/faq/qos.rst)
- [《Quality of Service (QoS) Rate Limiting》](https://github.com/openvswitch/ovs/blob/master/Documentation/howto/qos.rst)

Open vSwitch本身并不具备qos功能，是基于linux的"tc"功能实现的，是已经在linux内核中存在的功能。
而Open vSwitch所做的是对其部分支持的tc功能进行配置（因为Open vSwitch不是支持所有的tc功能）。
如果Open vSwitch不支持你需要的qos功能，那么可以直接使用linux的"tc"。

### 策略（Policing）
在linux的qos中，接收数据包使用的方法叫策略（policing），当速率超过了配置速率，就简单的把数据包丢弃。
不通过OpenFlow设置，直接在interface上设置。
[Frequently Asked Questions: Quality of Service (QoS)](https://github.com/openvswitch/ovs/blob/master/Documentation/faq/qos.rst) 中有一个例子：
``` bash
ovs-vsctl set interface vif1.0 ingress_policing_rate=10000
ovs-vsctl set interface vif1.0 ingress_policing_burst=8000
```
上面两行命令，把虚拟端口vif1.0的最大接收速率设置为10000kbps，桶大小设置为8000kb。
策略使用了简单的令牌桶（token bucket）算法。
我们以一定的速度不断生成令牌，除非令牌桶装满。
每接收一个包，需要消耗一个令牌；如果没有令牌了，就会把新到达的包丢弃。
（这里用“到达”和“接收”来区别，到达节点的包和其中被接收转发的包。)
如果到达包的速度大于令牌的生成速度，那么令牌很快消耗干净，新到达的包只能丢弃，那么接收包的速度很快就降下来，和令牌的生成速度一致。
所以接收包的速度依赖于令牌的生成速度，换句话说，不能大于令牌的生成速度，也就是最大接收速率，即`ingree_policing_rate`的值，单位是kbps。
如果到达包的速度小于令牌的生成速度，那么令牌很快堆满令牌桶，这时到达包的速度突然增大，令牌桶中有足够的令牌。这一瞬间可供消耗的令牌有桶中的令牌，也有不断生成的令牌，导致接收包的速度也会突然增大，大于令牌的生成速度，也就是大于我们设置的最大接收速率，称为突发接收速率。
这时虽然突发接收速率大于最大接受速率，但是也是有限制的，最多增加的速率（最大突发接收速率减去最大接收速率）依赖于桶的大小，换句话说，增加的吞吐量不能大于桶的大小，毕竟桶中令牌只有这么多（多余的可供消耗的令牌），即`ingress_policing_burst`的值，单位是kb。
在上面的例子中，如果所有包的大小都是1kb，那么最多增加的速率达到8000kbps，最大突发接收速率达到18000kbps。

注意：要实现ingress policing，内核必须支持NET_CLS_BASIC，NET_SCH_INGRESS，和NET_ACT_POLICE等模块，而NET_CLS_POLICE不需要，因为已经过时。

### 整形（Shaping）
在linux的qos中，发送数据包使用的方法叫整形（shaping）。
与策略的不同之处在于，它使用了队列（queue)，除了丢弃数据包之后，还可以缓存数据包延迟发送，或者调度改变数据包的发送顺序。
比策略更加精确和有效。
[Frequently Asked Questions: Quality of Service (QoS)](https://github.com/openvswitch/ovs/blob/master/Documentation/faq/qos.rst) 中有一个例子：
``` bash
ovs-vsctl -- \
  add-br br0 -- \
  add-port br0 eth0 -- \
  add-port br0 vif1.0 -- set interface vif1.0 ofport_request=5 -- \
  add-port br0 vif2.0 -- set interface vif2.0 ofport_request=6 -- \
  set port eth0 qos=@newqos -- \
  --id=@newqos create qos type=linux-htb \
      other-config:max-rate=1000000000 \
      queues:123=@vif10queue \
      queues:234=@vif20queue -- \
  --id=@vif10queue create queue other-config:max-rate=10000000 -- \
  --id=@vif20queue create queue other-config:max-rate=20000000

ovs-ofctl add-flow br0 in_port=5,actions=set_queue:123,normal
ovs-ofctl add-flow br0 in_port=6,acitons=set_queue:234,normal
```
上面的命令，分别把虚拟端口vif1.0，vif2.0的最大发送速率设置为10000000bps和20000000bps。
第2行，建立了一个网桥br0。
这里的`--`指当前的命令，即`ovs-vsctl`，是一种省略写法，当然也可以拆开为`ovs-vsctl`的多条命令。后面接`--`，实际是接着下一行，使第3行形成一个完整的命令。
第3行，把物理网卡端口eth0加入到网桥br0中。
第4、5行，将虚拟端口vif1.0和vif2.0加入到网桥br0中，并分别设置OpenFlow端口id为5和6。
第6行，设置eth0的qos规则为"newqos"，这里的`@`可以理解为变量或者指针，"newqos"这时还没有创建，接下来几行是创建它。
第7行，用`--id=@`开头，表示创建这个变量或者指针，赋值给它的值是后面语句的返回值。
后面的`create qos`表示创建了qos规则，将这个qos规则赋值给"newqos"，相当于把这个qos规则命名为"newqos"。
这个qos规则的类型是"linux-htb"。

qos规则有两个重要属性，分别是type和queues。
"tc"中，队列（queue）分为无类队列，有类队列。
无类队列只有一条队列，只有一种队列规则（qdisc）；而有类队列分为很多类（class），数据包到达时，根据不同的数据包类型，源目的ip，端口等等属性，被筛选器（filter）划分进不同的类中，不同的类可能有不同的队列规则，不同的类也可以继续划分，嵌套下去。
type就相当于不同的队列，具有不同的队列规则；queues就相当于有类队列的不同类。

这里的type值设置为"linux-htb"。
linux-htb使用了"tc"的htb队列（hieratchical token bucket），分层次的令牌桶队列，属于有类队列。
在无类队列中，最简单的是pfifo_fast队列，采取先入先出的算法，只能延迟数据包发送或丢弃数据包，不能对数据包进行调度，即改变数据包发送顺序。
还有一种tbf队列（token bucket filter），采取上面提到的令牌桶的算法，而htb就是在tbf的基础上修改为了有类队列，其核心算法还是令牌桶算法。
Open vSwitch的qos规则除了提供linux-htb类型，还提供了linux-hfsc类型，对应了"tc"中的hsfc队列（hieratchical fair service curve），分层次的公平服务曲线队列，它同时除了针对带宽，还针对延迟对数据包进行调度，其原理参考http://linux-ip.net/articles/hfsc.en 。

第8行，设置了"newqos"的一个额外属性，max-rate，表示最大发送速率，和上面的最大接收速率类似，其值为1000000000bps。
第9、10行，在qos规则中建立了两个queue，分别为vif10queue和vif20queue，key分别为123和234。
同样使用了`@`，即在第11、12行，创建了queue，并设置了最大发送速率分别为10000000bps和20000000bps，命名为vif10queue和vif20queue。
这两个queue属于属性queues，可以理解成htb算法中不同的类，它们的队列规则不同之处在于最大发送速率不同。
第14、15行，为网桥br0添加OpenFlow规则，当数据包用OpenFlow端口id为5的端口（即vif1.0）传递时，使用队列123（即vif10queue）发送；数据包用OpenFlow端口id为6（即vif2.0）传递时，使用队列234（即vif20queue）发送。
前面的命令全是创建qos规则，但是并没有使用，这两句才是使用qos规则的命令。
如果没有这两句命令，数据包发送一直使用默认的queue，没有对发送速率的限制。

linux-htb类型qos规则为queue提供了四种属性，上面只用到了max-rate，除此之外，还有min-rate，burst，priority。
- min-rate：最低发送速率，保障了最低带宽。
- burst：桶大小，和policing部分介绍的含义一样，和突峰发送速率相关。
- priority：优先级，数字越小，优先级越高，默认值为0。数据包发送时，发送优先级高的类里的数据包。

> 注意：openflow规则的priority值越大，优先级越高；而queue的prority值越小，优先级越高。

而linux-hfsc的queue只有max-rate和min-rate两种属性。

同样在整形里面，实际的最大发送速率大于我们设置的最大发送速率，因为有突峰发送速率存在。桶大小和最大突峰发送速率的关系，参看上一节策略部分。

Open vSwitch和qos功能相关的命令和属性并不多，所以如果无法满足需求，只能直接使用linux的"tc"功能。

### 实践

现在来完成一个整形（shaping）的实践：
``` bash
#创建网桥br0
vs-vsctl add-br br0

#创建两个虚拟端口p0和p1
ovs-vsctl add-port br0 p0 -- set interface p0 type=internal -- set interface p0 ofport_request=10
ovs-vsctl add-port br0 p1 -- set interface p1 type=internal -- set interface p1 ofport_request=11

#把以太网卡eth0加入到网桥br0中
ovs-vsctl add-port br0 eth0

#设置ip、网关、路由信息等，使br0能连上外网
ifconfig eth0 0
ifconfig br0 10.18.129.162/16 up
route add default gw 10.18.0.254
route add -net 169.254.0.0 netmask 255.255.0.0 dev br
#ip地址10.18.129.162/16是eth0之前的ip地址，现在将其设置到br0上
#默认网关10.18.0.254和路由169.254.0.0也分别是eth0之前的信息，现在将其设置到br0上
#另外还要把br0的ip地址设为空，因为不能和br0的ip地址冲突
#如果该环境是单网卡服务器，最好使用脚本，因为在设置过程中会造成ssh连接断开

#设置两个虚拟端口的ip分别为10.18.200.10/16和10.18.200.11/16
ifconfig p0 10.18.200.10/16 promisc up
ifconfig p1 10.18.200.11/16 promisc up

#清楚现在eth0上qos规则，所有qos规则及queue信息
#如果是第一次运行，可以不运行这两句命令
ovs-vsctl clear port eth0 qos
ovs-vsctl -- --all destroy QoS -- --all destroy Queue

#该qos规则创建两个queue，其最大发送速率为100mbps和200mbps，id为123和234
ovs-vsctl set port eth0 qos=@newqos -- \
--id=@newqos create qos type=linux-htb \
        other-config:max-rate=2000000000 \
        queues:123=@p0queue \
        queues:234=@p1queue -- \
--id=@p0queue create queue other-config:max-rate=100000000 -- \
--id=@p1queue create queue other-config:max-rate=200000000

#清楚br0上所有用ip匹配的openflow规则
#如果之前没有类似规则，可以不运行这句命令
ovs-ofctl del-flows br0 ip

#在br0上添加两条openflow规则，将源地址为10.18.200.10的ip数据包放入id为123的queue，将源地址为10.18.200.11的ip数据包放入id为234的queue
ovs-ofctl add-flow br0 priority=5,ip,nw_src=10.18.200.10,actions=set_queue:123,normal
ovs-ofctl add-flow br0 priority=5,ip,nw_src=10.18.200.11,actions=set_queue:234,normal
#注意：在nw_src/nw_dst/nw_proto等匹配规则前必须加上ip或者icmp关键字
#必须确认数据包的网络层协议类型才能使用网络层的源地址/目的地址/协议编号匹配，否则不生效
#同样，在tp_src/mod_tp_dst等匹配规则前也必须加上tcp/udp/sctp等关键字
#必须确认数据包的传输层协议类型才能使用传输层的源地址/目的地址匹配，否则不生效
#openflow规则不生效的很有可能的一种原因是其他规则的优先级更高，导致优先匹配了其他规则，即使不生效的规则匹配精度更高。
```

现在开始发包测试，利用发包测速工具[iperf](iperf.fr) 将数据包发往另一个节点（10.18.129.163）：
``` bash
#10.18.129.163
iperf -s -p 12345 -i 1

#10.18.129.162
iperf -c 10.18.129.163 -p 12345 -i 1 -t 30 -B 10.18.200.10 & \
iperf -c 10.18.129.163 -p 12345 -i 1 -t 30 -B 10.18.200.11
```

10.18.129.163上显示：
``` bash
------------------------------------------------------------
Server listening on TCP port 12345
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.18.129.163 port 12345 connected with 10.18.200.10 port 12345
[  5] local 10.18.129.163 port 12345 connected with 10.18.200.11 port 12345
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0- 1.0 sec  11.9 MBytes   100 Mbits/sec
[  5]  0.0- 1.0 sec  23.8 MBytes   200 Mbits/sec
[  4]  1.0- 2.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5]  1.0- 2.0 sec  23.8 MBytes   200 Mbits/sec
[  4]  2.0- 3.0 sec  11.9 MBytes  99.8 Mbits/sec
[  5]  2.0- 3.0 sec  23.8 MBytes   200 Mbits/sec
[  5]  3.0- 4.0 sec  23.8 MBytes   199 Mbits/sec
[  4]  3.0- 4.0 sec  11.9 MBytes   100 Mbits/sec
[  4]  4.0- 5.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5]  4.0- 5.0 sec  23.8 MBytes   199 Mbits/sec
[  4]  5.0- 6.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5]  5.0- 6.0 sec  23.8 MBytes   200 Mbits/sec
[  4]  6.0- 7.0 sec  11.9 MBytes  99.9 Mbits/sec
[  5]  6.0- 7.0 sec  23.8 MBytes   199 Mbits/sec
[  4]  7.0- 8.0 sec  11.9 MBytes  99.8 Mbits/sec
[  5]  7.0- 8.0 sec  23.8 MBytes   200 Mbits/sec
[  4]  8.0- 9.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5]  8.0- 9.0 sec  23.8 MBytes   200 Mbits/sec
[  4]  9.0-10.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5]  9.0-10.0 sec  23.8 MBytes   200 Mbits/sec
[  5] 10.0-11.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 10.0-11.0 sec  11.9 MBytes   100 Mbits/sec
[  4] 11.0-12.0 sec  11.9 MBytes  99.5 Mbits/sec
[  5] 11.0-12.0 sec  23.7 MBytes   199 Mbits/sec
[  4] 12.0-13.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5] 12.0-13.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 13.0-14.0 sec  11.9 MBytes  99.7 Mbits/sec
[  5] 13.0-14.0 sec  23.7 MBytes   199 Mbits/sec
[  5] 14.0-15.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 14.0-15.0 sec  11.9 MBytes   100 Mbits/sec
[  4] 15.0-16.0 sec  11.9 MBytes  99.5 Mbits/sec
[  5] 15.0-16.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 16.0-17.0 sec  11.9 MBytes  99.7 Mbits/sec
[  5] 16.0-17.0 sec  23.7 MBytes   199 Mbits/sec
[  4] 17.0-18.0 sec  11.9 MBytes   100 Mbits/sec
[  5] 17.0-18.0 sec  23.8 MBytes   200 Mbits/sec
[  5] 18.0-19.0 sec  23.8 MBytes   199 Mbits/sec
[  4] 18.0-19.0 sec  11.9 MBytes  99.6 Mbits/sec
[  4] 19.0-20.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5] 19.0-20.0 sec  23.8 MBytes   199 Mbits/sec
[  4] 20.0-21.0 sec  11.9 MBytes  99.7 Mbits/sec
[  5] 20.0-21.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 21.0-22.0 sec  11.9 MBytes  99.8 Mbits/sec
[  5] 21.0-22.0 sec  23.7 MBytes   199 Mbits/sec
[  4] 22.0-23.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5] 22.0-23.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 23.0-24.0 sec  11.9 MBytes   100 Mbits/sec
[  5] 23.0-24.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 24.0-25.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5] 24.0-25.0 sec  23.8 MBytes   199 Mbits/sec
[  4] 25.0-26.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5] 25.0-26.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 26.0-27.0 sec  11.9 MBytes   100 Mbits/sec
[  5] 26.0-27.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 27.0-28.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5] 27.0-28.0 sec  23.8 MBytes   199 Mbits/sec
[  4] 28.0-29.0 sec  11.9 MBytes  99.6 Mbits/sec
[  5] 28.0-29.0 sec  23.8 MBytes   200 Mbits/sec
[  4] 29.0-30.0 sec  11.9 MBytes  99.7 Mbits/sec
[  5] 29.0-30.0 sec  23.8 MBytes   200 Mbits/sec
[  5]  0.0-30.2 sec   718 MBytes   200 Mbits/sec
[  4]  0.0-30.3 sec   361 MBytes  99.7 Mbits/sec
```
可以看到最后的qos结果非常好。
节点10.18.129.162的两个进程使用两个不同的ip，占用两个不同的queue同时向节点10.18.129.163发包，接收方接收到数据包的带宽分别为200Mbps和99.7Mbps，符合之前发送方设置的200Mbps和100Mbps的最大发送速率。

* 注：Open vSwitch官方文档的github地址在不断变动，上述超链接可能失效，请在http://openvswitch.org 和https://github.com/openvswitch/ovs 查找。
 
** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" /> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>

