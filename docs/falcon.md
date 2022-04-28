## dpvs for falcon

#### collect dpvs counter

```shell
$sudo govs -s -type falcon
net.if.in.ring.drop.pkts 2194560
net.if.in.packets 18201983 iface=port0
net.if.in.bytes 1092335734 iface=port0
net.if.in.errors 0 iface=port0
net.if.in.dropped 5970028 iface=port0
net.if.out.packets 8 iface=port0
net.if.out.bytes 648 iface=port0
net.if.out.errors 0 iface=port0
net.if.in.packets 747 iface=port1
net.if.in.bytes 260090 iface=port1
net.if.in.errors 0 iface=port1
net.if.in.dropped 0 iface=port1
net.if.out.packets 16006472 iface=port1
net.if.out.bytes 960388488 iface=port1
net.if.out.errors 0 iface=port1
```

#### push dpvs counter to agent api(http://127.0.0.1:1988/v1/push)

```
# govs -s -type falcon | falcon_push
```

#### push counter to falcon by crontab

```
#crontab -e
* * * * * /usr/sbin/govs -s -type falcon | /usr/sbin/falcon_push
```
