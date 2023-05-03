# 安装Ubuntu桌面系统常用操作

以下操作仅适用于于个人使用习惯，包括一些个人常用但其他大部分人用不到的软件等。

## 安装

1. 选择最小安装。否则会安装很多不需要的东西，特别是游戏，还无法卸载。

2. 安装其他软件：不安装图形或无线硬件

3. 联网后再安装。这样安装时会下载语言和自带的输入法等，否则输入法问题很麻烦。

## 安装后操作

1. Startup Disk Creator

   由于是最小安装，所以“启动盘创建器”工具不会安装，可以在应用商店搜索“Startup Disk Creator”安装即可。

2. wireguard

   ```sh
   $ sudo apt install wireguard resolvconf # 如果不存在 resolvconf 可以先安装它 sudo apt install resolvconf
   $ scp tx:/mnt/share/node/ubuntu/system/etc/wireguard/wg0.conf .
   $ sudo mv wg0.conf /etc/wireguard
   $ systemctl enable wg-quick@wg0 --now
   ```

3. 挂载NFS

   ```sh
   $ sudo apt-get install nfs-common nfs-kernel-server # 如果挂载失败，则至少要安装 nfs-common
   $ scp tx:/mnt/share/archive/system/linux/etc/systemd/system/auto_mount_nfs.service .
   $ sudo mv auto_mount_nfs.service /etc/systemd/system/
   $ sudo systemctl enable auto_mount_nfs.service --now
   ```

4. NFS 相关权限设置

   由于需要在不同系统（Mac和Linux）上挂载相同的 NFS 目录，所以如果不进行设置会出现 Mac 上创建的文件无法在 Linux 上修改，Linux 创建的 Mac 也无法修改，所以需要将 Linux 用户添加到 Mac 用户所属的组。

   ```sh
   $ umask 0002 # Ubuntu 默认是该值，如果不是建议设置为该值
   $ usermod -aG dialout $(whoami) # 组名不一定是 dialout，只要是组 ID 为 501 即可
   ```

   > Mac 中用户组的组ID为`501`，而 Ubuntu 中，ID 为`501`的用户组为`dialout`，所以直接将当前用户附加到改组即可，如果不存在则创建一个 ID 为 `501` 的组来使用即可。

5. 信任自签名证书

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

6. 远程ssh登陆

   ```sh
   $ sudo apt-get install openssh-server
   ```

7. flameshot（火焰截图）

   应用商店(snap)

8. Edge

   官方下载安装包进行安装

9. qqmusic

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

10. dbeawer-ce

   [官网](https://dbeaver.io/download/)下载安装`Linux Debian package`包安装.

11. draw.io

    https://github.com/jgraph/drawio-desktop/releases

12. WPS

   [官网](https://www.wps.cn/product/wpslinux)下载安装

11. Lens

    参考[官网](https://docs.k8slens.dev/getting-started/install-lens/)

12. MarkText

    一般用Typore

13. curl

    ```shell
    $ sudo apt install curl -y
    ```

14. 安装KVM并安装win10

    安装KVM

    ```shell
    # 如果加上 --no-install-recommends 参数不会安装 virt-viewer 之类的包，这些包在非图形化界面用不到
    $ sudo apt install \
    qemu-kvm `# 一个提供硬件仿真的开源仿真器和虚拟化包，自动安装libvirt-clients – 一组客户端的库和API，用于从命令行管理和控制虚拟机和管理程序`\
    virtinst `# 一套为置备和修改虚拟机提供的命令行工具`\
    virt-manager `# 一款通过 libvirt 守护进程，基于 QT 的图形界面的虚拟机管理工具,自动安装 libvirt-daemon-system – 为运行 libvirt 进程提供必要配置文件的工具`\
    bridge-utils `# 一套用于创建和管理桥接设备的工具`
    ```

    安装后可以通过`sudo systemctl status libvirtd`查看虚拟化守护进程是否已经运行，如果没有则可以启动`sudo systemctl enable --now libvirtd`

    另外，请将当前登录用户加入 kvm 和 libvirt 用户组，以便能够创建和管理虚拟机。

    ```
    $ sudo usermod -aG kvm $USER && sudo usermod -aG libvirt $USER
    ```

    安装安成后需要重启系统，否则会出现无法连接的情况（可能由于权限不够，因为上面加入的用户还未生效）

15. containerd

    参见K8s相关笔记

16. 常见开发工具

    1. [jenv](https://github.com/jenv/jenv) - java多版本切换工具，还可以添加本地jdk进行切换。

    2. [gvm](https://github.com/moovweb/gvm) - golang多版本切换

       ```sh
       $ sudo apt install -y bison gcc make
       $ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
       ```

    3. n - nodejs多版本切换

17. 状态栏显示网速

    在Ubuntu22.04下暂不能通过`apt`安装，所以直接从源码安装：

    ```sh
    $ git clone https://github.com/fossfreedom/indicator-sysmonitor.git
    $ cd indicator-sysmonitor
    $ pip3 install psutil
    $ sudo make install
    $ cd .. && rm -rf indicator-sysmonitor
    $ nohup indicator-sysmonitor &
    ```

    > 安装后，通过鼠标打开设置修改监控属性以及开机启动。

18. 安装sz/rz

    ```sh
    # sz/rz 工具本身
    $ sudo apt install lrzsz
    # 自带的ssh不支持，所以需要安装zssh
    $ sudo apt install zssh
    # 使用zssh登陆远程(和ssh用法一样)
    $ zssh root@ip -p port
    ```

    使用：

    ```sh
    # 下载文件(远程sz，则本地rz)
    $ sz remote.file
    ## 进入等待后按ctrl + @键进入zssh终端，输入 rz 即可接收文件（windows terminal/tmux不支持。建议使用cygwin64）
    $ rz
    
    # 上传文件(本地rz，则远程sz)
    $ rz
    ## 进入等待后按ctrl+@，进入zssh终端，输入 sz <filename> 即可接收文件
    $ sz local.file
    ```

19. 重新安装`Vim`

20. 解决普通用户无法使用`1024`以下端口，参考[原文](https://my.oschina.net/lenglingx/blog/5603925)。

    ```sh
    #临时生效
    sysctl net.ipv4.ip_unprivileged_port_start=0
    
    #永久生效
    echo "net.ipv4.ip_unprivileged_port_start=0" >> /etc/sysctl.conf
    sysctl -p
    ```

21. 解决合上盖子时自动休眠问题

    将`2`文件中`HandleLidSwitch`和`HandleLidSwitchExternalPower`配置项值修改为`lock`或`ignore`，修改后重启生效。

    ```
    # 合上盖子时动作: suspend-暂停/休眠 ignore-无动作 lock-锁屏
    HandleLidSwitch=lock
    # 合上盖子时动作（外接电源时）: suspend-暂停/休眠 ignore-无动作 lock-锁屏
    HandleLidSwitchExternalPower=lock
    ```

## Ubuntu下常见问题解决

1. IDEA无法输入中文

   参考[网上文章](https://blog.csdn.net/weixin_43627118/article/details/120663214)，在Idea启动配置文件添加一行`-Drecreate.x11.input.method=true`

2. 应用商店无法打开

   如果挂载了`/home`目录，则可能是由于历史数据导致，删除`/home/username/snap/`目录即可。

