[WireGuard官网](https://www.wireguard.com/)

## 常用命令

```shell
# 不中断活跃连接的情况下重新加载配置文件
$ wg syncconf wg0 <(wg-quick strip wg0)
```

## 参考文档

- https://gobomb.github.io/post/wireguard-notes/
- <https://blog.mygallop.cn/2022/06/centos/wireguard-install/>
- [wg-quick 路由策略解读](https://icloudnative.io/posts/linux-routing-of-wireguard/)
- [Wireguard 使用笔记](https://gobomb.github.io/post/wireguard-notes/)