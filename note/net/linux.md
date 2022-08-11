## 查看网络连接

```shell
$ networkctl list 
IDX LINK      TYPE     OPERATIONAL SETUP    
  1 lo        loopback n/a         unmanaged
  2 wlp0s20f3 wlan     n/a         unmanaged
  3 docker0   bridge   n/a         unmanaged

3 links listed. 
```

创建网络命名空间

```shell
$ sudo ip netns add ns0
```

在指定命名空间网络下执行命令

```shell
# ip netns exec <网络命名空间> <命令>
$ ip netns exec ns0 ip add
```

## 一键脚本

https://github.com/abidwaqar/virtualLoadBalancer.git

```shell
#!/bin/bash

#创建两个网络命名空间
sudo ip netns add I16-0229-1
sudo ip netns add I16-0229-2

#创建两个 veth 对 | Creating veth pairs
sudo ip link add veth1 type veth peer name br-veth1
sudo ip link add veth2 type veth peer name br-veth2

#创建一个 Linux 网桥
sudo ip link add br1 type bridge

#使用已创建的 veth 对将两个命名空间与桥接连接 | Associating namespaces with bridge using veth pairs

#region 设置桥接和 veth 配对，以确保它们正常工作

#setting veth front end with namespaces
sudo ip link set veth1 netns I16-0229-1
sudo ip link set veth2 netns I16-0229-2

#setting veth backend with bridge
sudo ip link set br-veth1 master br1
sudo ip link set br-veth2 master br1

#Setting up bridge and veth pairs up
sudo ip link set br1 up
sudo ip link set br-veth1 up
sudo ip link set br-veth2 up
sudo ip netns exec I16-0229-1 ip link set veth1 up
sudo ip netns exec I16-0229-2 ip link set veth2 up

#Assigning IP addresses
sudo ip netns exec I16-0229-1 ip addr add 192.168.1.11/24 dev veth1
sudo ip netns exec I16-0229-2 ip addr add 192.168.1.12/24 dev veth2
sudo ip addr add 192.168.1.10/24 brd + dev br1

#endregion

#将 IP 地址和 ping 4 数据包从一个命名空间到另一个命名空间。使用 ping -c 4 将 ping 限制为四个数据包。（数据包丢失应为 0% | ping second namespace
sudo ip netns exec I16-0229-1 ping 192.168.1.12 -c 4

#在计算机上的命名空间和 iptable 规则中添加默认路由，以启用与 Internet 的通信
sudo ip -all netns exec ip route add default via 192.168.1.10
sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j MASQUERADE
#启用 ipv4 转发
sudo sysctl -w net.ipv4.ip_forward=1
sudo ip netns exec I16-0229-1 ping 8.8.8.8 -c 4

sudo apt update
sudo apt install resolvconf
sudo echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
sudo echo 'nameserver 8.8.4.4' >> /etc/resolv.conf
sudo systemctl start resolvconf.service

#将 4 数据包从两个命名空间 Ping 到 8.8.8.8。（丢包率应为零）
#从两个网络命名空间 Ping google.com（这是通过配置 resolv.conf 文件完成的）
sudo ip netns exec I16-0229-1 ping google.com -c 4
sudo ip netns exec I16-0229-2 ping google.com -c 4

#删除 iptables 规则、命名空间、veth 对和 linux 桥接
sudo iptables -t nat -D POSTROUTING -s 192.168.1.0/24 -j MASQUERADE
sudo ip netns del I16-0229-1
sudo ip netns del I16-0229-2
sudo ip link delete veth1
sudo ip link delete veth2
sudo ip link set br1 down
sudo brctl delbr br1
```

