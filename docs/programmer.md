## api

## dpdk mbuf struct

```
struct rte_mbuf *m
   +
   |          +                                                                +
   |          | <-------------------+ m->buf_len  +--------------------------> |
   |          |                                                                |
   |          |             +                                        +         |
   v          | m->data_off |  <-------+  m->data_len  +---------->  |         |
              |             |                                        |         |
   +---------------------------------------------------------------------------+
   |          |             |                                        |         |
   | rte_mbuf | headroom    |               data                     | tailroom|
   |          |             |                                        |         |
   +----------+-------------+----------------------------------------+---------+

              ^
              |
              +
        m->buf_addr
```


## sysctl

```c
struct vs_lcore_params_io {
	sturct {
		/* dev mbuf -> io rx queue */
		uint64_t nic_queues_iters[VS_MAX_NIC_RX_QUEUES_PER_IO_LCORE];
		uint64_t nic_queues_pkts[VS_MAX_NIC_RX_QUEUES_PER_IO_LCORE]; /* rte_eth_rx_burst() */

		/* io rx queue -> worker ring */
		uint64_t rings_iters[VS_MAX_WORKER_LCORES];
		uint64_t rings_pkts[VS_MAX_WORKER_LCORES];
		uint64_t rings_drop_iters[VS_MAX_WORKER_LCORES];
		uint64_t rings_drop_pkts[VS_MAX_WORKER_LCORES];
		uint64_t rings_drop_count[VS_MAX_WORKER_LCORES];
	} rx;
	sturct {
		/* Stats */
		uint64_t nic_ports_iters[VS_MAX_NIC_TX_PORTS_PER_IO_LCORE];
		uint64_t nic_ports_pkts[VS_MAX_NIC_TX_PORTS_PER_IO_LCORE]; /* rte_eth_tx_burst(port, io.tx.mbuf_out[port]) */
		uint64_t nic_ports_drop_iters[VS_MAX_NIC_TX_PORTS_PER_IO_LCORE];
		uint64_t nic_ports_drop_pkts[VS_MAX_NIC_TX_PORTS_PER_IO_LCORE];

	} tx;

	struct {
		struct rte_kni *kni[VS_MAX_NIC_TX_PORTS_PER_IO_LCORE];
		struct interface_stats stats[VS_MAX_NIC_TX_PORTS_PER_IO_LCORE];
	} kni;
};

struct vs_lcore_params_worker {
	/* stats */
	uint64_t		rings_in_iters[VS_MAX_IO_LCORES];
	uint64_t		rings_in_pkts[VS_MAX_IO_LCORES];
	uint64_t		rings_in_miss[VS_MAX_IO_LCORES];
	uint64_t		rings_in_miss_count[VS_MAX_IO_LCORES];
	uint64_t		rings_out_iters[VS_MAX_NIC_PORTS];
	uint64_t		rings_out_pkts[VS_MAX_NIC_PORTS];
	uint64_t		rings_out_drop_iters[VS_MAX_NIC_PORTS];
	uint64_t		rings_out_drop_pkts[VS_MAX_NIC_PORTS];
	struct ipv4_vs_stats	stats;				/* from ip_vs */
	uint64_t		estats[VS_EXT_STAT_LAST];

};

```

#### ip vs

struct ipv4_vs_service {
	struct ipv4_vs_stats stats;	/* Use per-cpu statistics for the service */
};
struct ipv4_vs_dest {
	struct ipv4_vs_stats stats;	/* Use per-cpu statistics for destination server */
};

## atomic


## eth stats
	struct rte_eth_stats *stats;
			rte_eth_stats_get(port, &stats);
			printf("I/O RX %u in (NIC port %u): NIC drop ratio = %.2f avg burst size = %.2f\n",
			     lcore, (unsigned)port,
			     (double)stats.imissed / (double)(stats.imissed + stats.ipackets),
			     ((double)lp->rx.nic_queues_count[i])/((double)lp->rx.nic_queues_iters[i]));
			lp->rx.nic_queues_iters[i] = 0;
			lp->rx.nic_queues_count[i] = 0;

## arp

get arp from /proc/net/arp

- fullnat: keepalived will get all rs arp
- dsnat: dest arp tabel miss -> tx from kni -> get arp
  * first pkg will time out
  * must enable sysctl net.ipv4.ip_forward



客户端发出第一个报文(m1)，ｄｐｖｓ　修改报文内容后，回复给客户端，该过程无状态
回复报文的ｓｅｑ初始值由 m1 的　４元组　＋　ｍ１　的ｓｅｑ，当前系统启动时间，以及ｍ１的ｔｃｐ　ｏｐｔｓ　算出
- [S]	c ->	vs
- [S.]	c <-	vs
	ipv4_vs_in(m1)
		ipv4_vs_synproxy_syn_rcv(m1)	// send m1 direct
			syn_proxy_reuse_skb	// get/set tcp_opt(TCPOPT_MSS,TCPOPT_WINDOW,TCPOPT_TIMESTAMP,TCPOPT_SACK_PERM


如果步骤一来自正常的客户端请求，客户端收到第一个报文的回复后，会回应对应的　ａｃｋ　报文，通过校验
- [.]	c ->	vs
- [S]		vs ->	rs
	ipv4_vs_in(m2)
    		pp->conn_schedule(m2)->tcp_conn_schedule(m2)->tcp_vs_conn_schedule(m2)->ipv4_vs_synproxy_ack_rcv(m2)
			// enqueue m2
			ipv4_vs_schedule(m2) -> ipv4_vs_conn_new(m2) -> skb_queue_tail(&cp->ack_skb, m2)
			// alloc syn_skb send to rs
			syn_proxy_send_rs_syn(m2, ...) // 向　ｒｓ　发送　ｓｙｎ　报文后，连接状态被置为　
				cp->syn_skb = rte_pktmbuf_clone(syn_skb, cp->svc->lp->mbuf_pool);

/*
 * Filter out-in ack packet when cp is at SYN_SENT state.
 * DROP it if not a valid packet, STORE it if we have
 * space left.
 */
- [.]   c ->    vs
	ipv4_vs_in(m2.x)
		//queue_tail(&cp->ack_skb, m2.x) if  len(sck_skb) < 3(sysctl_ipv4_vs_synproxy_skb_store_thresh)
		ipv4_vs_synproxy_filter_ack(m2.x) -> skb_queue_tail(&cp->ack_skb, m2.x);

开启 reuse_cl, tw, fw, cw, la 开关后，可以在收到　ａｃｋ　报文后，重现发送　ｓｙｎ　给　ｒｓ　来重用连接
- [.]   c ->    vs // reuse conn
	ipv4_vs_in(m2.x)
		ipv4_vs_synproxy_reuse_conn(m2.x)
			// clean cp->ack_skb cp->syn_skb
			__syn_proxy_reuse_conn(m2.x)
			// alloc syn_skb send to rs
			syn_proxy_send_rs_syn(m2, ...)
				cp->syn_skb = rte_pktmbuf_clone(syn_skb, cp->svc->lp->mbuf_pool);

/*
 * Syn-proxy step 3 logic: receive syn-ack from rs
 * Update syn_proxy_seq.delta and send stored ack skbs
 * to rs.
 */
- [S.]		vs <-	rs
- [.]		vs ->	rs
	ipv4_vs_in(m3)
	  handle_response(m3, ...)
	    ipv4_vs_synproxy_synack_rcv(m3, ...) // drop m3, and send cp->ack_skb to rs
		
## checksum 

- 计算原理
  - IP校验和只校验 IP头部。
  - UDP校验和校验 UDP头部和UDP数据部分。
  - TCP校验和校验 TCP头部和TCP数据部分。
- 计算原理：
  - 将所有位划分为16位(2字节)的字。
  - 将所有16位的字相加，得到一个32位的字。
  - 将32位字的高16位与低16位相加，直到高16位全部为0。得到一个16位的字。
  - 将3中得到的字，取反即为校验和	


[sack](https://www.ibm.com/developerworks/cn/linux/l-tcp-sack/)

