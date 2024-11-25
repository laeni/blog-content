# Install

`apt install resolvconf -y`

# README

`/usr/share/doc/resolvconf/README.gz`

## 其他

1. Resolvconf 根据`nameserver`所关联的接口的名称进行排序。（更确切地说，根据存储它们的记录名称对它们进行排序，但惯例是记录的命名方式与它们所属的接口相同，可能带有一些后缀。） 特定顺序由 /etc/resolvconf/interface-order 文件确定。

## CTL

### 添加 DNS

```sh
echo -n "$RESOLVINFO" | /sbin/resolvconf -a "${IFACE}.${MYNAME}"
```

> 