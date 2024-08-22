# CentOS 6.9 升级 OpenSSH 到 8.9

```
yum update
yum install openssl-devel gcc perl make gmake g++ automake -y

curl -LO https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/openssh-8.9p1.tar.gz
tar -xf openssh-8.9p1.tar.gz
cd openssh-8.9p1
./configure # --prefix=/usr --sysconfdir=/etc/ssh # 如果此步报 openssl lib 无法找到相关的错误，可以卸载 openssl-devel 重装，实在不行可以重新编译安装 openssl
make
sudo make install
```

> 所有版本: [pub-OpenBSD-OpenSSH-portable安装包下载_开源镜像站-阿里云 (aliyun.com)](https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/)

XXX 系统下可以直接使用已编译的进行安装

```
curl https://iboxpay.oss-cn-shenzhen.aliyuncs.com/
```

