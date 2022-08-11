## 安装Ubuntu桌面系统常用操作

### 安装

1. 选择最小安装。否则会安装很多不需要的东西，特别是游戏，还无法卸载。

2. 不安装其他软件（可选）

3. 联网后再安装。这样安装时会下载语言和自带的输入法等，否则输入法问题很麻烦。

### 常用软件安装

1. Startup Disk Creator

   由于是最小安装，所以“启动盘创建器”工具不会安装，可以在应用商店搜索“Startup Disk Creator”安装即可。

2. 信任自签名证书

   由于有些服务器只是自己用，但是为了安全所以自己签发了证书，所以需要信任根证书。

   根证书已放在`/usr/local/share/ca-certificates/Laeni Global Root CA.crt`

   ```shell
   # 将根证书添加到系统信任列表（添加后可以用curl等命令）
   $ sudo update-ca-certificates
   # 将根证书添加到 nssdb 库（添加后可以用浏览器等使用nssdb的程序且一般需要重启对应程序才生效）
   $ sudo apt install libnss3-tools # 如果不安装该工具可能会提示 certutil 命令不存在
   $ certutil -A -d sql:/home/laeni/.pki/nssdb -t "C,," -n "Laeni Global Root CA" -i /usr/local/share/ca-certificates/Laeni\ Global\ Root\ CA.crt
   ```

   > 这里只添加了系统和nssdb，如果需要在java中使用，则需要再将根证书添加到java证书库中。

3. wireguard

   ```shell
   $ sudo apt install resolvconf # 官网没有该命令，但是如果该命令不存在时需要安装
   $ sudo apt install wireguard
   ```

4. 火焰截图

   应用商店(snap)

5. Edge

   官方下载安装包进行安装

6. qqmusic

   1. 安装libfuse2，否则可能报`dlopen(): error loading libfuse.so.2`

      ```shell
      $ sudo apt install libfuse2
      ```

   2. [官网](https://y.qq.com/download/download.html)下载`.AppImage`版本

   3. 制作启动脚本

      ```shell
      # 部分电脑（如华为）需要禁用才能启动
      $ echo './qqmusic-1.1.4.AppImage --disable-gpu-sandbox --no-sandbox' > qqmusic.sh
      ```

   4. 如果安装deb版本，安装完成后需要修改启动脚本`/usr/share/applications/qqmusic.desktop`

      修改启动命令为`Exec=/opt/qqmusic/qqmusic --disable-gpu-sandbox %U`
      修改后可能需要重启才生效

7. dbeawer-ce

   [官网](https://dbeaver.io/download/)下载安装`Linux Debian package`包安装.

8. draw.io

   [github](https://github.com/jgraph/drawio-desktop/releases)下载安装

9. WPS

   [官网](https://www.wps.cn/product/wpslinux)下载安装

10. Lens

11. MarkText

12. Notepad++

    应用商店(snap)

13. curl

    ```shell
    $ sudo apt install curl -y
    ```

14. 安装KVM并安装win10

    1. 安装KVM

       ```shell
       # 加上 --no-install-recommends 参数不会安装 virt-viewer 之类的包，这些包在非图形化界面用不到
       sudo apt install \
       qemu-kvm \     # 一个提供硬件仿真的开源仿真器和虚拟化包，自动安装libvirt-clients – 一组客户端的库和API，用于从命令行管理和控制虚拟机和管理程序
       virtinst \     # 一套为置备和修改虚拟机提供的命令行工具
       virt-manager \ # 一款通过 libvirt 守护进程，基于 QT 的图形界面的虚拟机管理工具,自动安装 libvirt-daemon-system – 为运行 libvirt 进程提供必要配置文件的工具
       bridge-utils   # 一套用于创建和管理桥接设备的工具
       ```

       安装后可以通过`sudo systemctl status libvirtd`查看虚拟化守护进程是否已经运行，如果没有则可以启动`sudo systemctl enable --now libvirtd && sudo systemctl start libvirtd`

       另外，请将当前登录用户加入 kvm 和 libvirt 用户组，以便能够创建和管理虚拟机。

       ```
       $ sudo usermod -aG kvm $USER
       $ sudo usermod -aG libvirt $USER
       ```

       安装安成后需要重启系统，否则会出现无法连接的情况（可能由于权限不够，因为上面加入的用户还未生效）

    2. 创建网桥

       KVM 会自己创建一个 virbr0 的桥接网络，但是这个是一个 NAT 的网络，没有办法跟局域网内的其他主机进行通信，所以还是自己建一个桥接网络。

       参考： https://www.leyeah.com/article/kvm-installation-handbook-ubuntu-668480

15. containerd

16. jenv

    java多版本切换工具，还可以添加本地jdk进行切换。

    gvm - golang多版本切换

    n - nodejs多版本切换

## Ubuntu下常见问题解决

1. IDEA无法输入中文

   参考[网上文章](https://blog.csdn.net/weixin_43627118/article/details/120663214)，在Idea启动配置文件添加一行`-Drecreate.x11.input.method=true`
