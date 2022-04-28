## 1 build environment

#### 1.1 install pkg depends
centos(>=7.0)

``` 
sudo yum install kernel-devel-`uname -r` kernel-headers-`uname -r`
sudo yum install libpcap-devel popt-devel bison ruby-devel openssl-devel golang cmake libgcrypt rubygems
sudo gem install json_pure md2man
```

ubuntu(16.04)

```
sudo apt-get install linux-headers-`uname -r`
sudo apt-get install libpcap-dev libpopt-dev ruby-dev cmake \
golang bison flex libssl-dev rpm
sudo gem update --system
sudo gem install json_pure md2man
```

#### 1.2 build package

```
cd vs
git submodule update --init --recursive
export RUBYOPT="-KU -E utf-8:utf-8"
cmake .
make
make package
```

if build package failed, you can use the following method to clean build things before rebuild.

```
./scripts/clean.sh
```

#### 1.3 xiaomi

使用公司内部装机系统安装centos 7.2, 启动后，切换到3.10.0-327.el7.x86_64内核

```
#先查看内核编号
[root@c1-lvs-10g-inter02:~]#awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (3.18.6-2.el7.centos.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)
2 : CentOS Linux (0-rescue-a8f42a178b5e4d8d9f7114e14108797e) 7 (Core)

#修改启动内核编号
[root@c1-lvs-10g-inter02 ~]# grub2-set-default 1

#查看启动内核编号
[root@c1-lvs-10g-inter02 ~]# grub2-editenv list
saved_entry=1
```

## 2 install package

centos

```
sudo rpm -ivh dpvs-xxxx.rpm
sudo yum install quagga
```

ubuntu

```
```


## 3 dpdk

#### 3.1 切换到root账号,将/usr/sbin加入环境变量PATH中

```
sudo -s
PATH=/usr/sbin:$PATH
```

#### 3.2 网卡 & 驱动
安装完成之后，先加载内核模块

```
modprobe rte_kni
modprobe igb_uio
```

将网卡绑定到uio模块上

```
[root@c1-lvs-10g-inter02 ~]# dpdk-devbind.py --status

Network devices using DPDK-compatible driver
============================================
<none>

Network devices using kernel driver
===================================
0000:01:00.0 'I350 Gigabit Network Connection' if=eth0 drv=igb unused=igb_uio *Active*
0000:01:00.1 'I350 Gigabit Network Connection' if=eth1 drv=igb unused=igb_uio 
0000:06:00.0 '82599ES 10-Gigabit SFI/SFP+ Network Connection' if=eth2 drv=ixgbe unused=igb_uio 
0000:06:00.1 '82599ES 10-Gigabit SFI/SFP+ Network Connection' if=eth3 drv=ixgbe unused=igb_uio 

Other network devices
=====================
<none>
```

将网卡设备eth2(06:00.0) eth3(06:00.1) 绑定到 igb_uio驱动上

```
[root@c1-lvs-10g-inter02 ~]# dpdk-devbind.py --bind=igb_uio 06:00.0
[root@c1-lvs-10g-inter02 ~]# dpdk-devbind.py --bind=igb_uio 06:00.1
[root@c1-lvs-10g-inter02 ~]# dpdk-devbind.py --status

Network devices using DPDK-compatible driver
============================================
0000:06:00.0 '82599ES 10-Gigabit SFI/SFP+ Network Connection' drv=igb_uio unused=
0000:06:00.1 '82599ES 10-Gigabit SFI/SFP+ Network Connection' drv=igb_uio unused=

Network devices using kernel driver
===================================
0000:01:00.0 'I350 Gigabit Network Connection' if=eth0 drv=igb unused=igb_uio *Active*
0000:01:00.1 'I350 Gigabit Network Connection' if=eth1 drv=igb unused=igb_uio 

Other network devices
=====================
<none>
```


恢复网卡设备06:00.0 到系统驱动上

```
[root@c1-lvs-10g-inter02 ~]# dpdk-devbind.py --bind=ixgbe 06:00.0
```

#### 3.4 Use of Hugepages in the Linux Environment

[dpdk guides](http://dpdk.org/doc/guides-16.04/linux_gsg/sys_reqs.html)

Reserving Hugepages for DPDK Use

```
echo 10240 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

#On a NUMA machine, pages should be allocated explicitly on separate nodes:
echo 5120 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
echo 5120 > /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
```

Using Hugepages with the DPDK

```
mkdir /mnt/huge
mount -t hugetlbfs nodev /mnt/huge
```

remount

```
umount -a -t hugetlbfs
mount -t hugetlbfs nodev /mnt/huge
```

The mount point can be made permanent across reboots, by adding the following line to the /etc/fstab file:

```
nodev /mnt/huge hugetlbfs defaults 0 0
```

For 1GB pages, the page size must be specified as a mount option:

```
nodev /mnt/huge_1GB hugetlbfs pagesize=1GB 0 0
```

#### 3.5 性能优化
通过grub配置指定huge page，并隔离用于dpck的cpu

[guides](http://dpdk.org/doc/guides-16.04/linux_gsg/nic_perf_intel_platform.html)

```
default_hugepagesz = 1G hugepagesz = 1G hugepages = 8
isolcpus = 1,2,3,4
```

## 4 dpvs example

以 c1-lvs-10g-inter02.bj 为例, 网卡的对应关系为
- 06:00.0 port0 eth2 lan
- 06:00.1 port1 eth3 wan

#### 4.1 env
| host | wan | vip | lan | local ip|
|--- | --- | ---| ---| ----|
|inter01.bj | eth3<br> 10.105.251.74/30 | eth3<br> 10.115.57.101/24<br> 10.115.57.102/24 | eth2<br> 10.115.42.1/24 | eth2<br> 10.115.42.100~200|
|inter02.bj | eth3<br> 10.105.251.78/30 | eth3<br> 10.115.57.101/24<br> 10.115.57.102/24 | eth2<br> 10.115.43.1/24 | eth2<br> 10.115.43.100~200|
|inter03.bj | eth3<br> 10.105.251.90/30 | eth3<br>  | eth2<br> 10.115.54.1/24 | eth2|
|inter04.bj | eth3<br> 10.105.251.94/30 | eth3<br>  | eth2<br> 10.115.55.1/24 | eth2|

#### 4.2 config(dpvs.conf)

```
dpvs {
	#wan
	# 指定port 1的ip/netmask, dpvs启动后，会虚拟一个网卡veth1对应到port 1上，并将ip设置到veth1上(ip addr add 10.105.251.78/30 dev veth1 )
	ifa 10.105.251.78/30 port 1; # 点对点出网
	# 设定port 1网卡的路由, 10.115.57.0/24 从 port1 出，下一跳ip为10.105.251.73, 出口ip缺省为10.105.251.74(一般不使用，fullnat会修改源地址)
	# 启动后，会将该路由添加到系统里 ip (ip route add 10.115.57.0/24 via 10.105.251.77)
	lpm 10.115.57.0/24 port 1 sip 10.105.251.78 tip 10.105.251.77; # vip从点对点出网
	lpm 10.105.251.0/24 port 1 sip 10.105.251.78 tip 10.105.251.77; # vip从点对点出网
	#lan 开启relay选项后，网卡绑定的ip数据报文会转发到kni网卡上，用来做keepalived健康检查
	ifa 10.115.43.100/24 port 0 relay;
	lpm 0.0.0.0/0 port 0 sip 10.115.43.100 tip 10.115.43.254;
	# rs 的路由
	lpm 10.115.52.0/24 port 0 sip 10.115.43.100 tip 10.115.43.254;
	lpm 10.115.54.0/24 port 0 sip 10.115.43.100 tip 10.115.43.254;
	lpm 10.115.55.0/24 port 0 sip 10.115.43.100 tip 10.115.43.254;

	#ctrl 使用的cpu核心
	ctrl_lcore "1";

	#rx "(PORT, QUEUE, LCORE, HASHTYPE), ...";
	#List of NIC RX ports and queues handled by the I/O RX lcores
	#HASHTYPE为网卡所在的类型 0:origin 1:wan 2:lan
	rx "(0,0,2,2),(1,0,2,1)";

	#tx "(PORT, LCORE, ...";
	#List of NIC TX ports handled by the I/O TX
	#tx/kni tx
	tx "(0,2),(1,2)";

	#worker 使用的cpu核心, worker 的个数必须是2的正整数幂(1, 2, 4, 8, 16, 32, ...)
	worker_lcores "4";

```



#### 4.3 usage

```
dpvs -l 1-4 -- -c /etc/dpvs/dpvs.conf
```

应该可以看到veth0(port0), veth1(port1) 在dpvs.conf里配置的ip和路由已被添加到系统

```shell
[yubo@c1-lvs-10g-inter02:~/dpvs/vs][master]$ip route
default via 10.115.56.254 dev eth0 
10.105.251.0/24 via 10.105.251.77 dev veth1 
10.105.251.76/30 dev veth1  proto kernel  scope link  src 10.105.251.78 
10.115.43.0/24 dev veth0  proto kernel  scope link  src 10.115.43.100 
10.115.52.0/24 via 10.115.43.254 dev veth0 
10.115.54.0/24 via 10.115.43.254 dev veth0 
10.115.55.0/24 via 10.115.43.254 dev veth0 
10.115.56.0/24 dev eth0  proto kernel  scope link  src 10.115.56.2 
10.115.57.0/24 via 10.105.251.77 dev veth1 
169.254.0.0/16 dev eth0  scope link  metric 1002

[yubo@c1-lvs-10g-inter02:~/dpvs/vs][master]$ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 6c:92:bf:30:03:ae brd ff:ff:ff:ff:ff:ff
    inet 10.115.56.2/24 brd 10.115.56.255 scope global eth0
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 6c:92:bf:30:03:af brd ff:ff:ff:ff:ff:ff
7: veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether e2:dc:16:1b:4c:9b brd ff:ff:ff:ff:ff:ff
    inet 10.115.43.100/24 scope global veth0
       valid_lft forever preferred_lft forever
8: veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 16:4d:65:f9:43:df brd ff:ff:ff:ff:ff:ff
    inet 10.105.251.78/30 scope global veth1
       valid_lft forever preferred_lft forever

```

see also `dpvs(8) ipvsadm(8)`

## 5 quagga

```
[root@c1-lvs-10g-inter02 quagga]# cp -f /usr/share/doc/quagga-0.99.22.4/zebra.conf.sample /etc/quagga/zebra.conf 
[root@c1-lvs-10g-inter02 quagga]# cp -f /usr/share/doc/quagga-0.99.22.4/ospfd.conf.sample /etc/quagga/ospfd.conf
```

```
[root@c1-lvs-10g-inter02 quagga]# cat ospfd.conf 
service password-encryption
debug ospf packet all

interface lo
interface veth0
interface veth1
 ip ospf network point-to-point
 ip ospf hello-interval 1
 ip ospf dead-interval 4

router ospf
 ospf router-id 10.105.251.78
 log-adjacency-changes
 auto-cost reference-bandwidth 100000
 network 10.105.251.76/30 area 0.0.0
 network 10.115.57.0/24 area 0.0.0.0
```


## 6 keepalived
see also `keepalived(8) keepalived.conf(5)`

```
[root@c1-lvs-10g-inter02 quagga]# cat /etc/keepalived/keepalived.conf 
local_address_group laddr_g1 {
    10.115.43.100-102
}

virtual_server_group dpvs_test {
    10.115.57.102 80
}

virtual_server group dpvs_test {
    delay_loop 7
    lb_algo wrr
    lb_kind FNAT
    hysteresis 0
    alpha
    omega
    quorum 1
    quorum_up "ip addr add 10.115.57.102/32 dev lo"
    quorum_down "ip addr del 10.115.57.102/32 dev lo"
    protocol TCP
    laddr_group_name laddr_g1
    syn_proxy

    real_server 10.115.54.1 80 {
        weight 100
        inhibit_on_failure
        TCP_CHECK {
            connect_timeout 4
        }
    }

    real_server 10.115.55.1 80 {
        weight 100
        inhibit_on_failure
        TCP_CHECK {
            connect_timeout 4
        }
    }
}

```


```
[root@c1-lvs-10g-inter02 dpvs]# service  keepalived start
Reloading systemd:                                         [  OK  ]
Starting keepalived (via systemctl):                       [  OK  ]
[root@c1-lvs-10g-inter02 dpvs]# chkconfig  keepalived on
```

## 7 icmp

dpvs 只对已经存在的连接做icmp应答，对于常用的ICMP_ECHOREPLY/ICMP_ECHO可通过kni转给kernel来处理

- 开启转发开关 sysctrl.conf (net.ipv4.ip_forward = 1)
- 设置好路由
- dsnat，还需用iptable/netfilter要做snat来应对tcp/udp之外的协议
  * `sudo iptables -t nat -A POSTROUTING -s 1.1.1.0/24 -o veth1  -j SNAT --to-source 1.1.2.103 #for not tcp/udp protocol`


## 8 其他系统配置
还需要将local address 绑定到内网网卡veth1上

```
ip addr add 11.115.43.100/24 dev veth1
ip addr add 11.115.43.101/24 dev veth1
ip addr add 11.115.43.102/24 dev veth1
```
