```shell
# ip link add name bridge_name type bridge 也可以简化 ip link add bridge_name type bridge
# ip link set bridge_name up
想要添加Interface到网桥上，interface状态必须是Up
# ip link set eth0 up
添加eth0 interface到网桥上
# ip link set eth0 master bridge_name
从网桥解绑eth0
# ip link set eth0 nomaster
eth0 可以关闭的
# ip link set eth0 down
删除网桥可以用
# ip link delete bridge_name type bridge
也可以简化为
# ip link del bridge_name
```

