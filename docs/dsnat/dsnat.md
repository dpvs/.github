## env

snat 是三层的服务，目前支持 tcp/udp/icmp(doing) 的原地址转发

| host		| wan			| vip			| lan			| local ip		|
|---------	| ---			| ---			| ---			| ---			|
|yubo9020	| port0 1.1.2.1/24	| port0 1.1.1.103/24(vs)| port1 1.1.1.1/24	| port1 1.1.2.103-104	|
|yubo990	| -			| -			| port1 1.1.2.2/24(rs)	| -			|
|vm		| port0 1.1.1.3/24	| -			| -			| -			|



#### dsnat
client:
ip route add 1.1.2.0/24  via 1.1.1.1
1.1.1.3(vm) -> 1.1.2.2:81 ---dsnat---> 1.1.2.103-104 -> 1.1.2.2

#### web server(yubo990)

```shell
sudo apt install nginx
#edit config
root@yubo-990:~# cat /etc/nginx/sites-available/default
server {
    listen 0.0.0.0:81 default_server;
    add_header Content-Type "text/plain;charset=utf-8";
    add_header Cache-Control 'private,max-age=0';
    add_header Expires '-1';
    return 200 "hostname=$hostname\r\nhttp_host=$http_host\r\nserver=$server_addr:$server_port\r\nclient=$remote_addr:$remote_port\r\n";
}
```


#### yubo9020

- [install dpvs](../quick_start.md)
- configure
  * [dpvs.conf](dpvs.conf)
  * [keepalived.conf](keepalived.conf)

dsnat
---

```
#add virtual service( src network/netmask )
ipvsadm -A -t 1.1.1.0/24 --dsnat

#add local address ( public ip pool)
ipvsadm -P -t 1.1.1.0/24 -z 1.1.2.103
ipvsadm -P -t 1.1.1.0/24 -z 1.1.2.104

# at vm
curl 1.1.2.2:81

```

#### other

###### icmp
- dsnat，还需用iptable/netfilter要做snat来应对tcp/udp之外的协议
  * `sudo iptables -t nat -A POSTROUTING -s 1.1.1.0/24 -o veth1  -j SNAT --to-source 1.1.2.103 #for not tcp/udp protocol`


