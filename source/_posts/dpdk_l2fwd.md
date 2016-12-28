title: DPDK-l2fwd详解
date: 2015-03-31 13:59:13
tags: 
- dpdk
categories: dpdk
toc: true
---

L2 forwarding sample application在DPDK（Data Plane Development Kit）的基础上实现了第二层（链路层）的数据包转发。

<!--more-->
**Title: [dpdk-l2fwd详解](http://aidaizyy.github.io/dpdk_l2fwd)**
**Author: [Yunyao Zhang（张云尧）](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-04-15](http://aidaizyy.github.io)**

## 概要

版本：DPDK-1.8.0

本例中实现了相邻端口之间的相互转发。
比如一共4个端口可用，那么端口1收到数据后会转发给端口2，端口2收到数据后会转发给端口1，端口3和端口4也会相互转发。

## 编译

设置环境变量
``` bash
export RTE_SDK=/(RTE_SDK) #DPDK的路径
export RTE_TARGET=x86_64-native-linuxapp-gcc #DPDK的编译目标
```

进入示例目录
``` bash
cd /(RTE_SDK)/example/l2wfd
```

编译
``` bash
make
```

## 运行

``` bash
./build/l2wfd [EAL options] -- -p PORTMASK [-q NQ -T t]
```

- EAL options
	- DPDK EAL的默认参数，必须参数为-c COREMASK -n NUM。
	- COREMASK：一个十六进制位掩码表示分配的逻辑内核数量。
	- NUM：一个十进制整数表示内存通道数量。

- -p PORTMASK
	PORTMASK：一个十六进制位掩码表示分配的端口数量。

- -q NQ
	NQ：表示分配给每个逻辑内核的收发队列数量。

- -T t
	t: 表示打印统计数据到屏幕上的时间间隔，默认为10秒。

``` bash
./build/l2fwd -c f -n 4 -- -q 4 -p ffff
```
表示，分配给4个逻辑内核，每个内核分别有4个收发队列，而一共分配了16个端口。

## 详解

### 初始化EAL(Environment Abstraciton Layer)
``` C
	ret = rte_eal_init(argc, argv);
	if (ret < 0)
		rte_exit(EXIT_FAILURE, "Invalid EAL arguments\n");
```

### 参数传递
``` C
	ret = l2fwd_parse_args(argc, argv);
	if (ret < 0)
		rte_exit(EXIT_FAILURE, "Invalid L2FWD arguments\n");
```
EAL参数传递已经在rte_eal_init()函数中完成了，这里主要传递“--”后面的参数。
传递参数之后，得到三个变量。
- l2fwd_enabled_port_mask：可用端口位掩码
- l2fwd_rx_queue_per_lcore：每个逻辑内核的收取队列数量
- timer_period：打印统计数据的时间间隔

### 创建内存池
``` C
	l2fwd_pktmbuf_pool =
		rte_mempool_create("mbuf_pool", NB_MBUF,
				   MBUF_SIZE, 32,
				   sizeof(struct rte_pktmbuf_pool_private),
				   rte_pktmbuf_pool_init, NULL,
				   rte_pktmbuf_init, NULL,
				   rte_socket_id(), 0);
	if (l2fwd_pktmbuf_pool == NULL)
		rte_exit(EXIT_FAILURE, "Cannot init mbuf pool\n");
```
- "mbuf_pool"：内存池的名称
- NB_MBUF：内存池中存储mbuf的数量
- MBUF_SIZE: mbuf的大小
- 32：内存池缓存的大小

### 端口处理
``` C
	//rte_eth_dev_count()函数返回端口总数
	nb_ports = rte_eth_dev_count();
	if (nb_ports == 0)
		rte_exit(EXIT_FAILURE, "No Ethernet ports - bye\n");

	if (nb_ports > RTE_MAX_ETHPORTS)
		nb_ports = RTE_MAX_ETHPORTS;
```
``` C
	for (portid = 0; portid < nb_ports; portid++) {
		//跳过未分配或者不可用端口
		if ((l2fwd_enabled_port_mask & (1 << portid)) == 0)
			continue;
	}
```
可用端口位掩码表示，左数第n位如果为1，表示端口n可用，如果左数第n位如果为0，表示端口n不可用。
要得到第x位为1还是0，我们的方法是将1左移x位，得到一个只在x位为1，其他位都为0的数，再与位掩码相与。结果为1，那么第x位为1，结果位0，那么第x位为0.

### 设置每个端口的目的端口
这里设置数据包进入端口后，转发给相邻的端口。
每两个端口为一对，相互转发。
``` C
	for (portid = 0; portid < nb_ports; portid++) {
		if ((l2fwd_enabled_port_mask & (1 << portid)) == 0)
			continue;

		if (nb_ports_in_mask % 2) {
			l2fwd_dst_ports[portid] = last_port;
			l2fwd_dst_ports[last_port] = portid;
		}
		else
			last_port = portid;

		nb_ports_in_mask++;

		rte_eth_dev_info_get(portid, &dev_info);
	}
	if (nb_ports_in_mask % 2) {
		printf("Notice: odd number of ports in portmask.\n");
		l2fwd_dst_ports[last_port] = last_port;
	}
```

### 初始化端口的配置信息
为每个端口分配到相应的逻辑内核
每个端口只对应一个逻辑内核
每个逻辑内核对应l2fwd_rx_queue_per_lcore个端口
``` C
	for (portid = 0; portid < nb_ports; portid++) {
		if ((l2fwd_enabled_port_mask & (1 << portid)) == 0)
			continue;

		//得到一个收取队列未分配满且可用的逻辑内核
		while (rte_lcore_is_enabled(rx_lcore_id) == 0 ||
		       lcore_queue_conf[rx_lcore_id].n_rx_port ==
		       l2fwd_rx_queue_per_lcore) {
			rx_lcore_id++;
			if (rx_lcore_id >= RTE_MAX_LCORE)
				rte_exit(EXIT_FAILURE, "Not enough cores\n");
		}

		if (qconf != &lcore_queue_conf[rx_lcore_id])
			/* Assigned a new logical core in the loop above. */
			qconf = &lcore_queue_conf[rx_lcore_id];

		qconf->rx_port_list[qconf->n_rx_port] = portid;
		qconf->n_rx_port++;
		printf("Lcore %u: RX port %u\n", rx_lcore_id, (unsigned) portid);
	}
```

### 初始化每个端口
``` C
	for (portid = 0; portid < nb_ports; portid++) {
		if ((l2fwd_enabled_port_mask & (1 << portid)) == 0) {
			printf("Skipping disabled port %u\n", (unsigned) portid);
			nb_ports_available--;
			continue;
		}

		printf("Initializing port %u... ", (unsigned) portid);
		fflush(stdout);
		//初始化端口，第二个参数和第三个参数表示分配收取队列和发送队列的数量
		ret = rte_eth_dev_configure(portid, 1, 1, &port_conf);
		if (ret < 0)
			rte_exit(EXIT_FAILURE, "Cannot configure device: err=%d, port=%u\n",
				  ret, (unsigned) portid);

		//得到端口对应的mac地址，存入l2fwd_ports_eth_addr[]数组
		rte_eth_macaddr_get(portid,&l2fwd_ports_eth_addr[portid]);

		fflush(stdout);
		//初始化一个收取队列，nb_rxd指收取队列的大小，最大能够存储mbuf的数量
		ret = rte_eth_rx_queue_setup(portid, 0, nb_rxd,
					     rte_eth_dev_socket_id(portid),
					     NULL,
					     l2fwd_pktmbuf_pool);
		if (ret < 0)
			rte_exit(EXIT_FAILURE, "rte_eth_rx_queue_setup:err=%d, port=%u\n",
				  ret, (unsigned) portid);

		fflush(stdout);
		//初始化一个发送队列，nb_txd指发送队列的大小，最大能够存储mbuf的数量
		ret = rte_eth_tx_queue_setup(portid, 0, nb_txd,
				rte_eth_dev_socket_id(portid),
				NULL);
		if (ret < 0)
			rte_exit(EXIT_FAILURE, "rte_eth_tx_queue_setup:err=%d, port=%u\n",
				ret, (unsigned) portid);

		//开始运行该端口
		ret = rte_eth_dev_start(portid);
		if (ret < 0)
			rte_exit(EXIT_FAILURE, "rte_eth_dev_start:err=%d, port=%u\n",
				  ret, (unsigned) portid);

		printf("done: \n");

		rte_eth_promiscuous_enable(portid);

		printf("Port %u, MAC address: %02X:%02X:%02X:%02X:%02X:%02X\n\n",
				(unsigned) portid,
				l2fwd_ports_eth_addr[portid].addr_bytes[0],
				l2fwd_ports_eth_addr[portid].addr_bytes[1],
				l2fwd_ports_eth_addr[portid].addr_bytes[2],
				l2fwd_ports_eth_addr[portid].addr_bytes[3],
				l2fwd_ports_eth_addr[portid].addr_bytes[4],
				l2fwd_ports_eth_addr[portid].addr_bytes[5]);

		//初始化端口的统计数据
		memset(&port_statistics, 0, sizeof(port_statistics));
	}
```

### 检查每个端口的连接状态
``` C
	check_all_ports_link_status(nb_ports, l2fwd_enabled_port_mask);
```

### 在每个逻辑内核上启动线程，开始转发
``` C
	rte_eal_mp_remote_launch(l2fwd_launch_one_lcore, NULL, CALL_MASTER);
	RTE_LCORE_FOREACH_SLAVE(lcore_id) {
		if (rte_eal_wait_lcore(lcore_id) < 0)
			return -1;
	}
```

收包
``` C
	for (i = 0; i < qconf->n_rx_port; i++) {

		portid = qconf->rx_port_list[i];
		//收包，一次最多收取MAX_PKT_BURST个数据包
		nb_rx = rte_eth_rx_burst((uint8_t) portid, 0,
					 pkts_burst, MAX_PKT_BURST);
		
		//更新统计数据
		port_statistics[portid].rx += nb_rx;

		for (j = 0; j < nb_rx; j++) {
			m = pkts_burst[j];
			rte_prefetch0(rte_pktmbuf_mtod(m, void *));
			//转发
			l2fwd_simple_forward(m, portid);
		}
	}
```

转发
替换源MAC地址和目的MAC地址
``` C
	static void
	l2fwd_simple_forward(struct rte_mbuf *m, unsigned portid)
	{
		struct ether_hdr *eth;
		void *tmp;
		unsigned dst_port;

		dst_port = l2fwd_dst_ports[portid];
		eth = rte_pktmbuf_mtod(m, struct ether_hdr *);

		//目的地址
		/* 02:00:00:00:00:xx */
		tmp = &eth->d_addr.addr_bytes[0];
		*((uint64_t *)tmp) = 0x000000000002 + ((uint64_t)dst_port << 40);

		//源地址
		ether_addr_copy(&l2fwd_ports_eth_addr[dst_port], &eth->s_addr);

		l2fwd_send_packet(m, (uint8_t) dst_port);
	}
```

将数据包推送至发送队列，如果发送队列存够MAX_PKT_BURST，即每次最大收取包的数量，就会发包
``` C
	static int
	l2fwd_send_packet(struct rte_mbuf *m, uint8_t port)
	{
		unsigned lcore_id, len;
		struct lcore_queue_conf *qconf;

		lcore_id = rte_lcore_id();

		qconf = &lcore_queue_conf[lcore_id];
		len = qconf->tx_mbufs[port].len;
		qconf->tx_mbufs[port].m_table[len] = m;
		len++;

		//当发包队列存够MAX_PKT_BURST，发包
		if (unlikely(len == MAX_PKT_BURST)) {
			l2fwd_send_burst(qconf, MAX_PKT_BURST, port);
			len = 0;
		}

		qconf->tx_mbufs[port].len = len;
		return 0;
	}
```

每隔一定时间也会发包
``` C
	//上次收包时间和这次收包时间差
	diff_tsc = cur_tsc - prev_tsc;
	//如果时间差大于我们设定的阈值，这里是100us
	if (unlikely(diff_tsc > drain_tsc)) {

		for (portid = 0; portid < RTE_MAX_ETHPORTS; portid++) {			
			if (qconf->tx_mbufs[portid].len == 0)
				continue;
			//发包
			l2fwd_send_burst(&lcore_queue_conf[lcore_id],
					 qconf->tx_mbufs[portid].len,
					 (uint8_t) portid);
					qconf->tx_mbufs[portid].len = 0;
		}
		
		if (timer_period > 0) {
				
			timer_tsc += diff_tsc;

			//如果累积时间超过我们设定的阈值，就打印出统计数据，默认是10s
			if (unlikely(timer_tsc >= (uint64_t) timer_period)) {

				//打印数据在发生在主逻辑内核上
				if (lcore_id == rte_get_master_lcore()) {
					//打印统计数据
					print_stats();
					//累积时间置零
					timer_tsc = 0;
				}
			}
		}

		prev_tsc = cur_tsc;
	}
```
这两种情况都会产生发包，无论是发送队列存够阈值MAX_PKT_BURST，或者，时间差超过阈值brain_tsc，都会把发送队列上MAX_PKT_BURST个数据包推送出去，如果不足MAX_PKT_BURST，则把发送队列上全部数据包推送出去。

发包函数
``` C
	static int
	l2fwd_send_burst(struct lcore_queue_conf *qconf, unsigned n, uint8_t port)
	{
		struct rte_mbuf **m_table;
		unsigned ret;
		unsigned queueid =0;

		m_table = (struct rte_mbuf **)qconf->tx_mbufs[port].m_table;
		//发包
		ret = rte_eth_tx_burst(port, (uint16_t) queueid, m_table, (uint16_t) n);
		//更新统计数据
		port_statistics[port].tx += ret;
		//丢包
		if (unlikely(ret < n)) {
			//更新统计数据
			port_statistics[port].dropped += (n - ret);
			do {
				//把丢包部分free掉
				rte_pktmbuf_free(m_table[ret]);
			} while (++ret < n);
		}

		return 0;
	}
```
在函数rte_eth_tx_burst()中：
- port：端口号。
- queueid：端口中的发送队列号。本例中每个端口都只有一个发送队列，所以固定为0。
- m_table：**rte_mbuf数据
- n：发送包的数量

** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" /> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>
