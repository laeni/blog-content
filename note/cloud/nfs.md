## 挂载NFS

### Mac

命令：`mount -o resvport <ip>:<NFS目录> <本地挂载目录>`

```shell
$ sudo mount -o resvport 10.10.1.1:/nfs nfs-test
```

