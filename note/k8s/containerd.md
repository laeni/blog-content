# 安装

- 通过`nerdctl`直接安装全部工具

  参考命令如下：

  ```shell
  $ curl -o  -L https://github.com/containerd/nerdctl/releases/download/v1.0.0/nerdctl-full-1.0.0-linux-amd64.tar.gz
  $ tar Cxzvvf /usr/local nerdctl-full-1.0.0-linux-amd64.tar.gz
  $ ln -s /usr/local/libexec/cni/* /usr/local/bin/
  # 开启自起：下载 https://github.com/containerd/containerd/blob/main/containerd.service 到 `/usr/local/lib/systemd/system/` 中
  $ sudo systemctl daemon-reload && sudo systemctl enable --now containerd
  ```
  
- 依次手动安装

  适用于不需要安装全部工具的情况使用。
  
  1. 安装containerd
     从 https://github.com/containerd/containerd/releases 下载二进制文件并解压到 `/usr/local/bin`
     
     通过 systemd 启动containerd并开机自启动：
     下载 https://github.com/containerd/containerd/blob/main/containerd.service 到 `/usr/local/lib/systemd/system/` 中
  
     ```shell
     $ systemctl daemon-reload
     $ systemctl enable --now containerd
     ```

  2. 安装 runc
  
     ```shell
     $ curl -LO ...... # 实际上好像不能直接下载
     $ install -m 755 runc.amd64 /usr/local/sbin/runc
     ```

  3. 安装 CNI 插件

     从 https://github.com/containernetworking/plugins/releases 下载存档，然后解压到`/opt/cni/bin`
  
     ```shell
     $ tar Cxzvf /usr/local/bin cni-plugins-linux-amd64-v1.1.1.tgz
     ```
  
  4. 安装nerdctl
  
     nerdctl是Docker命令兼容的工具， https://github.com/containerd/nerdctl/releases

## 无root模式

1. 启用cgroup-v2

   新版系统默认已经启用，但是还需要针对当前用户做一些配置，详情可以参考[官网](https://rootlesscontaine.rs/getting-started/common/cgroup2/#enabling-cgroup-v2)。

2. 启用无root

   ```shell
   $ containerd-rootless-setuptool.sh install
   [INFO] Checking RootlessKit functionality
   [INFO] Checking cgroup v2
   [WARNING] The cgroup v2 controller "cpu" is not delegated for the current user ("/sys/fs/cgroup/user.slice/user-1000.slice/user@1000.service/cgroup.controllers"), see https://rootlesscontaine.rs/getting-started/common/cgroup2/
   [INFO] Checking overlayfs
   [INFO] Requirements are satisfied
   [INFO] Creating "/home/laeni/.config/systemd/user/containerd.service"
   [INFO] Starting systemd unit "containerd.service"
   + systemctl --user start containerd.service
   + sleep 3
   + systemctl --user --no-pager --full status containerd.service
   ● containerd.service - containerd (Rootless)
        Loaded: loaded (/home/laeni/.config/systemd/user/containerd.service; disabled; vendor preset: enabled)
        Active: active (running) since Sat 2022-07-30 23:22:53 CST; 3s ago
      Main PID: 8282 (rootlesskit)
         Tasks: 32
        Memory: 18.2M
           CPU: 74ms
        CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/containerd.service
                ├─8282 rootlesskit --state-dir=/run/user/1000/containerd-rootless --net=slirp4netns --mtu=65520 --slirp4netns-sandbox=auto --slirp4netns-seccomp=auto --disable-host-loopback --port-driver=builtin --copy-up=/etc --copy-up=/run --copy-up=/var/lib --propagation=rslave /usr/local/bin/containerd-rootless.sh
                ├─8292 /proc/self/exe --state-dir=/run/user/1000/containerd-rootless --net=slirp4netns --mtu=65520 --slirp4netns-sandbox=auto --slirp4netns-seccomp=auto --disable-host-loopback --port-driver=builtin --copy-up=/etc --copy-up=/run --copy-up=/var/lib --propagation=rslave /usr/local/bin/containerd-rootless.sh
                ├─8308 slirp4netns --mtu 65520 -r 3 --disable-host-loopback --enable-sandbox --enable-seccomp 8292 tap0
                └─8316 containerd
   
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.879914284+08:00" level=error msg="failed to initialize a tracing processor \"otlp\"" error="no OpenTelemetry endpoint: skip plugin"
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.879937130+08:00" level=info msg="loading plugin \"io.containerd.grpc.v1.cri\"..." type=io.containerd.grpc.v1
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.879983060+08:00" level=info msg="Start cri plugin with config {PluginConfig:{ContainerdConfig:{Snapshotter:overlayfs DefaultRuntimeName:runc DefaultRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0} UntrustedWorkloadRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0} Runtimes:map[runc:{Type:io.containerd.runc.v2 Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[BinaryName: CriuImagePath: CriuPath: CriuWorkPath: IoGid:0 IoUid:0 NoNewKeyring:false NoPivotRoot:false Root: ShimCgroup: SystemdCgroup:false] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0}] NoPivot:false DisableSnapshotAnnotations:true DiscardUnpackedLayers:false IgnoreRdtNotEnabledErrors:false} CniConfig:{NetworkPluginBinDir:/opt/cni/bin NetworkPluginConfDir:/etc/cni/net.d NetworkPluginMaxConfNum:1 NetworkPluginConfTemplate: IPPreference:} Registry:{ConfigPath: Mirrors:map[] Configs:map[] Auths:map[] Headers:map[]} ImageDecryption:{KeyModel:node} DisableTCPService:true StreamServerAddress:127.0.0.1 StreamServerPort:0 StreamIdleTimeout:4h0m0s EnableSelinux:false SelinuxCategoryRange:1024 SandboxImage:k8s.gcr.io/pause:3.6 StatsCollectPeriod:10 SystemdCgroup:false EnableTLSStreaming:false X509KeyPairStreaming:{TLSCertFile: TLSKeyFile:} MaxContainerLogLineSize:16384 DisableCgroup:false DisableApparmor:false RestrictOOMScoreAdj:false MaxConcurrentDownloads:3 DisableProcMount:false UnsetSeccompProfile: TolerateMissingHugetlbController:true DisableHugetlbController:true DeviceOwnershipFromSecurityContext:false IgnoreImageDefinedVolumes:false NetNSMountsUnderStateDir:false EnableUnprivilegedPorts:false EnableUnprivilegedICMP:false} ContainerdRootDir:/var/lib/containerd ContainerdEndpoint:/run/containerd/containerd.sock RootDir:/var/lib/containerd/io.containerd.grpc.v1.cri StateDir:/run/containerd/io.containerd.grpc.v1.cri}"
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.880016746+08:00" level=info msg="Connect containerd service"
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.880036723+08:00" level=info msg="Get image filesystem path \"/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs\""
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.880042723+08:00" level=warning msg="Running containerd in a user namespace typically requires disable_cgroup, disable_apparmor, restrict_oom_score_adj set to be true"
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.880191858+08:00" level=warning msg="failed to load plugin io.containerd.grpc.v1.cri" error="failed to create CRI service: failed to create cni conf monitor for default: failed to create cni conf dir=/etc/cni/net.d for watch: mkdir /etc/cni/net.d: permission denied"
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.880255636+08:00" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.880278287+08:00" level=info msg=serving... address=/run/containerd/containerd.sock
   7月 30 23:22:53 laeni-PC containerd-rootless.sh[8316]: time="2022-07-30T23:22:53.880289834+08:00" level=info msg="containerd successfully booted in 0.051192s"
   + systemctl --user enable containerd.service
   Created symlink /home/laeni/.config/systemd/user/default.target.wants/containerd.service → /home/laeni/.config/systemd/user/containerd.service.
   [INFO] Installed "containerd.service" successfully.
   [INFO] To control "containerd.service", run: `systemctl --user (start|stop|restart) containerd.service`
   [INFO] To run "containerd.service" on system startup automatically, run: `sudo loginctl enable-linger laeni`
   [INFO] ------------------------------------------------------------------------------------------
   [INFO] Use `nerdctl` to connect to the rootless containerd.
   [INFO] You do NOT need to specify $CONTAINERD_ADDRESS explicitly.
   ```

   根据日志输出完成其他操作

   ```shell
   $ sudo loginctl enable-linger laeni
   ```

   > 运行无根容器时如果提示`exec: "newuidmap": executable file not found in $PATH`，需要`sudo apt install uidmap`。

3. 关闭无root模式

   ```shell
   $ containerd-rootless-setuptool.sh  uninstall
   [INFO] Unit buildkit.service is not installed
   [INFO] Unit containerd-fuse-overlayfs.service is not installed
   + systemctl --user stop containerd.service
   + systemctl --user disable containerd.service
   [INFO] Uninstalled "containerd.service"
   [INFO] This uninstallation tool does NOT remove containerd binaries and data.
   [INFO] To remove data, run: `/usr/local/bin/rootlesskit rm -rf /home/laeni/.local/share/containerd`
   ```

## 生成默认配置文件

由于[k8s文档](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E6%97%B6)要求我们先生成默认配置，以便后续用到，所以可以通过`containerd config default > /etc/containerd/config.toml`生成默认配置。

# CTL

有多个工具可以和`containerd`进行交互。

## ctr

// TODO

## crictl

专门针对 K8S 做优化，所以如果用 K8S 的话一般需要安装它（目前 kubeadm 必须使用它）。

## nerdctl

nerdctl 的目的是尽量让其 API 于 Docker 兼容，所以可以用它构建镜像（ctr 和 crictl 不提供镜像构建功能）。

### 常见问题

- `crictl`下载的镜像使用`ctr`和`nerdctl`无法找到

  由于`ctr`支持命名空间，但其他两个不支持，而对于`ctr`来说，`nerdctl`和`ctr`（默认）的命名空间是`default`，而`crictl`的命名空间是`k8s.io`，所以`ctr`通过明确指定命名空间为`k8s.io`即可操作`crictl`下载的镜像。
