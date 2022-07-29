## 配置虚拟不可变的IP地址

由于笔记本电脑更换网络后本级ip地址会发生变化，变化后K8S需要重新初始化，所以需要给电脑加一个不会变化的静态ip。

### 找到“127.0.0.1”所在的网卡名

![image-20211211152330036](/home/laeni/.config/Typora/typora-user-images/image-20211211152330036.png)

这里的网卡名为: lo

### 给目标网卡绑定IP

临时添加与删除（重启后会还原）

> 添加
>
> `ifconfig lo:0 10.0.0.1/12 up`
>
> 删除
>
> `ifconfig lo:0 10.0.0.1 down`

永久

> 添加配置
>
> ```
> echo '
> auto lo:0
> iface lo:0 inet static
> address 10.0.0.1
> netmask 255.0.0.0
> ' > /etc/network/interfaces
> ```
>
> 重启网络使之生效
>
> `/etc/init.d/networking restart`

*似乎可以使用 tunctl(uml-utilities)工具进行添加。*

