# 安装Ubuntu桌面系统常用操作

以下操作仅适用于于个人使用习惯，包括一些个人常用但其他大部分人用不到的软件等。

## 安装

### 22.04

1. 选择最小安装。否则会安装很多不需要的东西，特别是游戏，还无法卸载。

2. 安装其他软件：不安装图形或无线硬件

3. 联网后再安装。这样安装时会下载语言和自带的输入法等，否则输入法问题很麻烦。

### 24.04

1. 预装应用：默认集合。

2. 专用软件：

   - [x] 为图形和 Wi-Fi 安装第三方软件

     安装完成后，如果无法使用远程桌面，可以切换显卡驱动试试（目前华为笔记本使用 nvidia-535 无法使用远程桌面。切换为 nvidia-470 则可以正常使用）

   - [ ] 下载并安装对其他媒体格式的支持（需要联网才可以勾选，不过一般也不用勾选）

## 安装后操作

### 系统配置

1. Ibus 拼音输入法配置

   - 【首选项】-【拼音模式】-【云输入选项】

     - [x] 启用云输入
     
     云输入源: 百度
     
   - 【首选项】-【快捷键】

     删除"切换繁体/简体中文模式"快捷键（该快捷键不常用，并且会导致其他软件无法使用对应快捷键）

2. 启用用户命名空间

   ```sh
   echo 'kernel.unprivileged_userns_clone=1' | sudo tee /etc/sysctl.d/userns.conf
   ```

### 软件安装

1. 重新安装`Vim`

    ```sh
    sudo apt remove vim-common
    sudo apt install vim
    ```

    > Ubuntu预装的是vim tiny版本，可能会导致键盘操作与预期不同，通过安装vim full版本，可以在不切换兼容模式的情况下正常使用键盘。

2. wireguard

   ```sh
   $ sudo apt install wireguard # 如果不存在 resolvconf 可以先安装它 sudo apt install resolvconf
   $ scp tx:/mnt/share/node/ubuntu/system/etc/wireguard/wg0.conf .
   $ sudo mv wg0.conf /etc/wireguard
   $ sudo systemctl enable wg-quick@wg0 --now
   ```

3. 挂载NFS

   ```sh
   $ sudo apt-get install nfs-common nfs-kernel-server # 如果挂载失败，则至少要安装 nfs-common
   $ scp tx:/mnt/share/archive/system/linux/usr/lib/systemd/system/auto_mount_nfs.service .
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

5. 启用 GENOME 原生远程桌面

   1. “设置 - 共享 - 远程桌面”中启用。

   2. 开启锁屏状态下允许连接。

      1. 安装`gnome-browser-connector`

         - 编译安装

           ```sh
           git clone https://gitlab.gnome.org/nE0sIghT/gnome-browser-connector.git
           cd gnome-browser-connector/
           meson --prefix=/usr builddir # meson 可能没有安装 - sudo apt install meson
           meson install -C builddir
           ```

         - 从软件包安装 - Ubuntu-24.04

           ```sh
           sudo apt install gnome-browser-connector
           ```

           > 如果此方式无法安装则只能使用编译方式安装。

      2. 浏览器中转到[allow-locked-remote-desktop](https://extensions.gnome.org/extension/4338/allow-locked-remote-desktop/)扩展页，开启右侧的开关即可。

      > 推荐使用上述的`gnome-browser-connector`方式，除了上述方式还可以直接安装本地可视化工具`gnome-shell-extension-manager`（`sudo apt install gnome-shell-extension-manager`），但是使用`gnome-shell-extension-manager`可能无法搜索到需要的软件包（在网页上可以搜索到）。

   3. 必要时开启自动登录

      1. 设置中开启自动登录。
      2. 通过`seahorse`命令打开**密码和密钥**，将**登录**的解锁密码设置为空（在**登录**上右键即可设置）。

6. 信任自签名证书

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

7. 远程ssh登陆

   ```sh
   $ sudo apt-get install openssh-server
   ```

8. Snipaste（截图）

   ```shell
   # 可能需要安装 libfuse2 库
   sudo apt install libfuse2
   ```

   应用商店(snap)

9. Edge

   官方下载安装包进行安装

10. 关闭 IBus **Unicode 码位** 快捷键

    此快捷键不仅几乎用不到，还会占用 IDEA 快速切换大小写的快捷键。

    1. 使用命令打开 IBus 首选项。

       ```bash
       $ ibus-setup
       ```

    2.  在“表情符号”选项卡清空“Unicode 码位”快捷键。

11. dbeawer-ce

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

    精简版: https://extensions.gnome.org/extension/6952/rezmon/

    [GNOME 插件](https://extensions.gnome.org/extension/6682/astra-monitor/).

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

19. 解决普通用户无法使用`1024`以下端口，参考[原文](https://my.oschina.net/lenglingx/blog/5603925)。

    ```sh
    #临时生效
    sysctl net.ipv4.ip_unprivileged_port_start=0
    
    #永久生效
    echo "net.ipv4.ip_unprivileged_port_start=0" >> /etc/sysctl.conf
    sysctl -p
    ```

20. 解决合上盖子时自动休眠问题

    将`/etc/systemd/logind.conf`文件中`HandleLidSwitch`和`HandleLidSwitchExternalPower`配置项值修改为`lock`或`ignore`，修改后重启生效。

    ```
    # 合上盖子时动作: suspend-暂停/休眠 ignore-无动作 lock-锁屏
    HandleLidSwitch=lock
    # 合上盖子时动作（外接电源时）: suspend-暂停/休眠 ignore-无动作 lock-锁屏
    HandleLidSwitchExternalPower=lock
    ```

21. Data Integration（Kettle / Spoon）

    1. 安装

       [官网](https://www.hitachivantara.com/en-us/products/pentaho-plus-platform/data-integration-analytics/download-pentaho.html)

       ```sh
       cd ~/Workspace/.Applicathon/
       curl -LO https://proxy.laeni.cn/pdi-ce-9.4.0.0-343.zip
       unzip pdi-ce-9.4.0.0-343.zip 
       ```

    2. 解决 Unbuntu 打开报警告问题

       参考[官网](https://community.hitachivantara.com/discussion/libwebkitgtk-10-0-on-ubuntu-2204-lts)做如下处理。

       编辑`/etc/apt/sources.list`文件，添加：

       ```
       # Add entry to repository that contains the old lib
       deb http://cz.archive.ubuntu.com/ubuntu bionic main universe 
       
       # Alternative link if upper does not work
       deb http://mirrors.kernel.org/ubuntu bionic main universe
       ```

       然后安装包：

       ```
       sudo apt-get update
       # 直接 update 会报证书相关的错，需要信任错误中给出的证书
       sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3B4FE6ACC0B21F32
       sudo apt-get update
       sudo apt-get install libwebkitgtk-1.0-0
       sudo apt install libcanberra-gtk-module libcanberra-gtk3-module
       ```

    3. 制作启动图标

       ```sh
       cat <<EOF | sudo tee ~/Workspace/.Applicathon/data-integration/data-integration.desktop
       [Desktop Entry]
       Name=Data Integration
       Comment=Data Integration
       GenericName=Data Integration
       Exec=./spoon.sh %U
       Icon=./spoon.png
       Type=Application
       StartupNotify=true
       Categories=Programming;
       EOF
       sudo ln -s ~/Workspace/.Applicathon/data-integration/data-integration.desktop /usr/local/share/applications/
       ```

22. Remmina

    使用 Snap 安装后需要另外授权:

    ```sh
    sudo snap connect remmina:audio-record :audio-record
    sudo snap connect remmina:avahi-observe :avahi-observe
    sudo snap connect remmina:cups-control :cups-control
    sudo snap connect remmina:mount-observe :mount-observe
    sudo snap connect remmina:password-manager-service :password-manager-service
    sudo snap connect remmina:ssh-keys :ssh-keys
    sudo snap connect remmina:ssh-public-keys :ssh-public-keys
    ```

    > 以上命令为 Remmina 首次打开时的提示。

23. Konsole

    ```sh
    sudo apt install konsole
    sudo update-alternatives --config x-terminal-emulator # 修改默认终端（默认文件管理器不生效）
    ```

    > 能同时向多个会话输入内容。

24. 忽略 Mac 临时文件

    在`~/.zshrc`和`~/.bashrc`添加如下内容。

    ```sh
    # 忽略属性，防止 Mac 生成的压缩文件在Linux解压报警告（解压 Mac 生成的压缩文件时可以有效防止警告和产生大量无用的临时文件）
    alias tar="tar --no-xattrs --exclude='.DS_Store' --exclude='._*'"
    ```

25. 允许 async-profiler 在没有权限的情况下收集信息.

    ```sh
    echo 'kernel.perf_event_paranoid=1 ' >> /etc/sysctl.d/99-async-profiler.conf
    echo 'kernel.kptr_restrict=0'        >> /etc/sysctl.d/99-async-profiler.conf
    ```

    > IDEA 等工具有时候会用到，如果没有添加，则在使用过程中可能会提示添加。

## Ubuntu下常见问题解决

1. IDEA无法输入中文

   参考[网上文章](https://blog.csdn.net/weixin_43627118/article/details/120663214)，在Idea启动配置文件添加一行`-Drecreate.x11.input.method=true`

2. 应用商店无法打开

   如果挂载了`/home`目录，则可能是由于历史数据导致，删除`/home/username/snap/`目录即可。

## 常用工具

- Linux 启动盘制作工具 - `/opt/apps/.bin/AppImage/balenaEtcher-1.18.11-x64.AppImage`

# 安装服务器 Linux 后常用操作 - proxy

镜像: 应用镜像-Docker

系统: Alibaba Cloud Linux

## 数据迁移

```yaml
- /etc/wireguard/wg0.conf
- /mnt/
- /root/
- /opt/html/
- /data/docker/
- /data/
```

## 软件安装

1. Wireguard

   1. 软件安装

      ```bash
      sudo apt install wireguard
      ```

   2. 配置准备

      ```bash
      ssh old
      scp /etc/wireguard/wg0.conf p:/etc/wireguard/
      ```

   3. 开启流量转发

      ```bash
      # 将转发配置写入配置文件
      echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
      echo 'net.ipv6.conf.all.forwarding = 1' >> /etc/sysctl.conf
      echo 'net.ipv6.conf.default.forwarding = 1' >> /etc/sysctl.conf
      # 使配置生效
      sudo sysctl -p
      ```

      > 验证：
      >
      > ```bash
      > cat /proc/sys/net/ipv4/ip_forward
      > cat /proc/sys/net/ipv6/conf/all/forwarding
      > ```
      >
      > 上述命令输出`1`即表示已开启转发。

   4. 启动

      ```bash
      systemctl enable wg-quick@wg0
      systemctl start wg-quick@wg0.service
      ```

3. Docker

   参考官网[Docker Engine](https://docs.docker.com/engine/install/ubuntu/)。

   1. 添加存储库

      ```bash
      # Add Docker's official GPG key:
      sudo apt-get update
      sudo apt-get install ca-certificates curl
      sudo install -m 0755 -d /etc/apt/keyrings
      sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
      sudo chmod a+r /etc/apt/keyrings/docker.asc
      
      # Add the repository to Apt sources:
      echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      sudo apt-get update
      ```

   2. 安装

      ```bash
      apt -y install docker-ce
      ```

