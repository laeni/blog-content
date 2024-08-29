---
title: Docker使用笔记
author: 'Laeni'
tags: Kubernetes, Containerd, Docker
date: '2021-10-01'
updated: '2022-10-16'
---

## 安装Docker引擎

### Linux环境下安装

#### Ubuntu

参考官网文档: https://docs.docker.com/engine/install/ubuntu/

```sh
# 添加源
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 使当前用户可以直接使用 docker 命令（需要重新登录会话）
sudo usermod -aG docker $USER
```

##### 常见问题

1. Centos 8 下安装可能会报找不到 containerd 的错。
   错误原因：一般时由于当前安装的docker版本所依赖的 containerd 版本不存在
   解决方案：从 Centos 7 存储库下载指定版本的 containerd 手动安装即可

   ```sh
   $ wget https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-xxx.rpm
   $ yum -y install containerd.io-xxx.rpm
   ```

#### 从软件包安装

[官方文档](https://docs.docker.com/engine/install/debian/#install-from-a-package)

##### Deepin 安装示例

进入[某版本的软件包下载页面](https://download.docker.com/linux/debian/dists/buster/pool/stable/amd64/)，下载`containerd.io_xxx_amd64.deb`、`docker-ce-cli_xxx_debian-buster_amd64.deb`和`docker-ce_xxx~debian-buster_amd64.deb`三个包依次安装即可。

#### 阿里云服务器安装

官方文档: <https://help.aliyun.com/zh/ecs/use-cases/install-and-use-docker-on-a-linux-ecs-instance>

##### Ubuntu

```sh
# 更新软件包列表并安装所需依赖包
sudo apt update
sudo apt-get -y install ca-certificates curl

# 创建/etc/apt/keyrings目录，并下载Docker的官方GPG密钥到该目录
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 将Docker仓库添加到系统的软件源列表并更新软件包列表
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] http://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update

# 安装Docker
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```



#### 卸载

```sh
# 首先查看Docker版本
$ yum list installed | grep docker
docker-ce.x86_64  18.05.0.ce-3.el7.centos @docker-ce-edge

# 执行卸载
$ yum -y remove docker-ce.x86_64

# 删除存储目录
$ rm -rf /etc/docker
$ rm -rf /run/docker
$ rm -rf /var/lib/dockershim
$ rm -rf /var/lib/docker

# 如果发现删除不掉，需要先 umount，如
$ umount /var/lib/docker/devicemapper
```

#### 启动、停止

这里仅记录使用 [`systemd`](https://docs.docker.com/config/daemon/systemd/)管理系统启动的Linux。

```sh
$ sudo systemctl start docker   # 启动 - 一般情况安装后会自动启动
$ sudo systemctl enable docker  # 开机启动 - 一般情况安装后会自动设置开机启动

$ sudo systemctl daemon-reload  # 重新加载systemctl配置
$ sudo systemctl restart docker # 重启docker
```

## 常见安装后配置

###  [修改默认存储路径](https://blog.csdn.net/qq_37674858/article/details/81669082)

```sh
$ sudo vi /usr/lib/systemd/system/docker.service
# 在里面的EXECStart的后面增加后如下:
ExecStart=/usr/bin/dockerd --graph /data/docker/lib
```

### 修改Cgroup Driver为 systemd

查询Docker使用的驱动：

```sh
$ docker info
***
 Cgroup Driver: systemd
 Cgroup Version: 1
***
```

Docker在默认情况下使用的 Cgroup Driver 为 `cgroup`，而 Kubernetes 其实推荐使用 `systemd` 来代替 `cgroupfs`。

编辑 `/etc/docker/daemon.json` (没有该文件就新建一个），添加如下启动项参数并重启即可：

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

### 修改镜像地址

参见[daemon.json](#daemon.json)。

### daemon.json

`daemon.json`为Docker系统级全局配置，路径为`/etc/docker/daemon.json` (没有该文件就新建一个）。每次修改后需要执行`systemctl daemon-reload && systemctl restart docker`使新配置生效。以下为常用的配置：

```json
{
    "registry-mirrors": ["https://mirror.ccs.tencentyun.com"],
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-opts": {"max-size":"100m", "max-file":"5"}
}
```

> **registry-mirrors:** 镜像加速地址，可以配置多个。
>
> **exec-opts:** 设置Cgroup Driver。
>
> **log-opts:** 容器日志选项，如果不配置则永不删除。`max-size`表示单个日志文件最大大小，超过将会被切分，`max-file`表示最多保留的切分后的日志数量。

## Docker Help

```
Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options{
    --config string      客户端配置文件的位置 (default"C:\\Users\\DELL\\.docker")
  -c, --context string     用于连接守护程序的上下文的名称(覆盖 DOCKER_HOST env var and default context set with "docker context use")
  -D, --debug              启用调试模式
  -H, --host list          要连接的守护程序套接字
  -l, --log-level string   设置日志记录级别("debug"|"info"|"warn"|"error"|"fatal")(default "info")
    --tls                使用 TLS; implied by --tlsverify
    --tlscacert string   信任证书仅由此CA签署 (default "C:\\Users\\DELL\\.docker\\ca.pem")
    --tlscert string     TLS证书文件的路径 (default "C:\\Users\\DELL\\.docker\\cert.pem")
    --tlskey string      TLS密钥文件的路径 (default "C:\\Users\\DELL\\.docker\\key.pem")
    --tlsverify          使用 TLS 并验证远程
  -v, --version            打印版本信息并退出
}
Management Commands(管理命令){
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers{
    // 修改容器为自启动
    docker container update --restart=always 容器id
  }
  context     Manage contexts
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks{
    Usage:  docker stack [OPTIONS] COMMAND

    Options:
          --orchestrator string   编排器使用 (swarm|kubernetes|all) | Orchestrator to use (swarm|kubernetes|all)

    Commands:
      deploy      部署新堆栈或更新现有堆栈 | Deploy a new stack or update an existing stack{
        请指定一个捆绑文件（带有--bundle-file）或一个Compose文件（带有--compose-file） | Please specify either a bundle file (with --bundle-file) or a Compose file (with --compose-file).

        # 使用示例
        docker stack deploy --compose-file xxx.yml <stack-name>
        docker stack deploy -c xxx.yml <stack-name>
        // 部署verdaccio
        docker stack deploy -c stack.yml verdaccio
      }
      ls          列表栈 | List stacks
      ps          列出堆栈中的任务 | List the tasks in the stack{
        Usage:  docker stack ps [OPTIONS] STACK

        Options:
          -f, --filter filter         Filter output based on conditions provided
              --format string         Pretty-print tasks using a Go template
              --no-resolve            Do not map IDs to Names
              --no-trunc              Do not truncate output
              --orchestrator string   Orchestrator to use (swarm|kubernetes|all)
          -q, --quiet                 Only display task IDs

        示例/*
          docker stack ps zookeeper
        */
      }
      rm          Remove one or more stacks
      services    列出堆栈中的服务 | List the services in the stack{
        docker stack services <stack-name>
        // 查看verdaccio
        docker stack services verdaccio
      }

    Run 'docker stack COMMAND --help' for more information on a command.

    示例/*
      docker stack deploy -c stack.yml zookeeper
    */
  }
  swarm       管理集群 | Manage Swarm{
    Usage:	docker swarm COMMAND

    Manage Swarm

    Commands:
      ca          显示并旋转根证书 | Display and rotate the root CA
      init        初始化 集群 | Initialize a swarm
      join        将节点和/或管理者加入集群 | Join a swarm as a node and/or manager
      join-token  管理加入令牌 | Manage join tokens
      leave       离开集群 | Leave the swarm
      unlock      解锁集群 | Unlock swarm
      unlock-key  管理解锁密钥 | Manage the unlock key
      update      更新集群 | Update the swarm

    Run 'docker swarm COMMAND --help' for more information on a command.
  }
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes
}
Commands(常规命令){
  images      # 列出本地主机上的镜像 | List images
  run{        # 在新容器中运行命令 | Run a command in a new container
    Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

    # 示例
    docker run alpine echo "Hello world"{    # 运行一个Linux发新版并在其中执行命令
      docker          Docker 的二进制执行文件。
      run            与前面的 docker 组合来运行一个容器。
      alpine          指定要运行的镜像，Docker首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像(alpine为Linux发行版,只有不到6MB大小)。
      echo "Hello world"    在启动的容器里执行的命令
    }

    Options:
    -t, --tty{                         # 分配一个pseudo-TTY(能使容器保持在后台) | Allocate a pseudo-TTY
      docker run -t alpine            # 让alpine在后台运行
    }
    -i, --interactive{                 # 保持STDIN打开，即使没有连接(使本地终端和容器能进行交互) | Keep STDIN open even if not attached
      docker run -i -t alpine            # 让alpine在后台运行的同时能够和alpine交互
    }
    -w, --workdir string               # 容器内的工作目录 | Working directory inside the container
      --expose list                    # 公开一个或多个端口 | Expose a port or a range of ports
      --name string                    为容器分配一个名称 | Assign a name to the container
    -p, --publish list{                将容器内部使用的网络端口映射到我们使用的主机上 | Publish a container's port(s) to the host'
      // 将宿主机8081端口映射到容器的80端口
      docker run -p 8081:80
      // 映射端口的同时监听IP,0.0.0.0表示监听全部
      docker run -p 0.0.0.0:8081:80
      // 映射端口的同时绑定TCP协议
      docker run -p 8081:80/TCP

      以上方式可以组合使用
    }
    -d, --detach                       在后台运行容器并打印容器ID | Run container in background and print container ID
      --restart string{                重启策略以在容器退出时应用（默认为“否”） | Restart policy to apply when a container exits (default "no")
        --restart always   # 自动启动
      }
    -v, --volume list{                 绑定挂载卷 | Bind mount a volume
      // 将"I:/docker"挂载到"/data"下
      docer run -v I:/docker:/data

      # 示例
      docker run --rm -v c:/Users:/data alpine ls /data
    }
      --rm                             当容器退出时自动删除它 | Automatically remove the container when it exits
    -e, --env list                     设置环境变量 | Set environment variables

      --add-host list                  Add a custom host-to-IP mapping(host:ip)
    -a, --attach list                  Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight),between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
    -c, --cpu-shares int               CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
      --env-file list                  Read in a file of environment variables
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
    -h, --hostname string              Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
    -l, --label list                   Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g.,92:d0:c6:0a:29:33)
    -m, --memory bytes                 Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)'
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --privileged                     赋予此容器扩展的特权 | Give extended privileges to this container
    -P, --publish-all                  Publish all exposed ports to random ports
      --read-only                      Mount the container's root filesystem as read only'
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "15")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
      --ulimit ulimit                  Ulimit选项(默认[]) | Ulimit options (default [])
    -u, --user string                  Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
      --volume-driver string           容器的可选卷驱动程序 | Optional volume driver for the container
      --volumes-from list              从指定容器装入卷 | Mount volumes from the specified container(s)
  }
  ps{         # 列表容器(列出所有容器)List containers
    docker ps -a                  # 列出所有容器的详细信息
    docker ps -a -q                # 列出所有容器的ID(CONTAINER ID)
  }
  stop{       # 停止一个或多个正在运行的容器 | Stop one or more running containers
    docker stop 6e2cfdbda7a0          # 停止容器ID为"6e2cfdbda7a0"的容器(可同时停止多个,并且可以使用容器名代替容器ID)
    docker stop $(docker ps -a -q)        # 停止所有容器(先通过ps查询所有容器)
  }
  rm{         # 删除一个或多个容器 | Remove one or more containers
    docker rm  $(docker ps -a -q)        # 删除所有容器
    docker rm $(docker stop $(docker ps -q))  # 停止并删除所有容器
  }
  rmi{        # 删除一个或多个图像 | Remove one or more images
    docker rmi xxx          # 删除id为"xxx"的镜像
    docker rmi $(docker images -q)  # 删除全部镜像
  }
  export{     # 将容器的文件系统导出为tar存档文件 | Export a container's filesystem as a tar archive'
    // 将id为"7355fcd2aa36"的容器导出为归档文件"mysql.container.tar.gz"
    docker export -o mysql.container.tar.gz 7355fcd2aa36
  }
  build{      # 从 Dockerfile 构建镜像 | Build an image from a Dockerfile
    Usage:  docker build [OPTIONS] PATH | URL | -

    示例{
      docker build ./{            // 使用"Dockerfile"文件从当前目录构建镜像
        ./ - Dockerfile 文件所在目录，可以指定 Dockerfile 的绝对路径
      }
      docker build -t runoob/centos:6.7 .{  // 指定新镜像的名字和tag
          -t - 指定要创建的目标镜像名
      }
    }

    Options:
    -t, --tag list             Name和可选的"Name:tag"格式的标记 | Name and optionally a tag in the 'name:tag' format

      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --cgroup-parent string    Optional parent cgroup for the container
      --compress                Compress the build context using gzip
      --cpu-period int          Limit the CPU CFS (Completely Fair
                  Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair
                  Scheduler) quota
    -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
    -f, --file string             Name of the Dockerfile (Default is
                  'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
    -m, --memory bytes            Memory limit
      --memory-swap bytes       Swap limit equal to memory plus swap:
                  '-1' to enable unlimited swap
      --network string          Set the networking mode for the RUN
                  instructions during build (default "default")
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of
                  the image
    -q, --quiet                   Suppress the build output and print image
                  ID on success
      --rm                      Remove intermediate containers after a
                  successful build (default true)
      --security-opt strings    Security options
      --shm-size bytes          Size of /dev/shm
      --target string           Set the target build stage to build.
      --ulimit ulimit           Ulimit options (default [])
  }
  logs{       # 获取容器的日志 | Fetch the logs of a container
    // 查看id为"b4ce4d2d7cb6"的容器日志
    docker logs b4ce4d2d7cb6
  }
  rename{     # 重命名容器 | Rename a container
    docker rename docker rename 912a2f001f4a graphic-auth-huanxun graphic-auth-huanxun
      912a2f001f4a - 容器ID或标签
  }
  restart     # 重新启动一个或多个容器 | Restart one or more containers


  attach      将本地标准输入，输出和错误流附加到正在运行的容器 | Attach local standard input, output, and error streams to a running container
  commit      根据容器的编号创建新镜像 | Create a new image from a container's changes'
  cp          在容器和本地文件系统之间复制文件/文件夹 | Copy files/folders between a container and the local filesystem{
    docker cp --help{
      Usage:	docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
        docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

      Copy files/folders between a container and the local filesystem

      Use '-' as the source to read a tar archive from stdin
      and extract it to a directory destination in a container.
      Use '-' as the destination to stream a tar archive of a
      container source to stdout.

      Options:
        -a, --archive       Archive mode (copy all uid/gid information)
        -L, --follow-link   Always follow symbol link in SRC_PATH
    }
  }
  create      创建一个新容器
  diff        检查容器文件系统上文件或目录的更改 | Inspect changes to files or directories on a container's filesystem'
  events      Get real time events from the server
  exec        Run a command in a running container
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     返回有关Docker对象的低级信息 | Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        从tar存档或STDIN加载镜像 | Load an image from a tar archive or STDIN{
    docker load --help{
      Usage:	docker load [OPTIONS]

      Load an image from a tar archive or STDIN

      Options:
        -i, --input string   Read from tar archive file, instead of STDIN
        -q, --quiet          Suppress the load output
    }
  }
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  pull        从仓库源中提取镜像或存储库 | Pull an image or a repository from a registry
  push        推送镜像或存储库到仓库源 | Push an image or a repository to a registry
  save        将一个或多个镜像保存到tar存档(默认情况下流式传输到 STDOUT) | Save one or more images to a tar archive (streamed to STDOUT by default){
    docker save --help{
      Usage:	docker save [OPTIONS] IMAGE [IMAGE...]

      Save one or more images to a tar archive (streamed to STDOUT by default)

      Options:
        -o, --output string   Write to a file, instead of STDOUT
    }
    示例{
      // 导出全部镜像
      IMAGES=$(docker images --format '{{.Repository}}:{{.Tag}}')
      for element in ${IMAGES[@]}
      do
        echo "saving ${element} ..."
        docker save ${element} >> allinone.tar
        echo "${element} saved"
      done

    }
  }
  search      在Docker Hub中搜索镜像 | Search the Docker Hub for images
  start       启动一个或多个已停止的容器 | Start one or more stopped containers
  stats       显示容器资源使用情况统计信息的实时流 | Display a live stream of container(s) resource usage statistics
  tag{        # 为镜像添加标签 | Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
    docker tag 860c279d2fec runoob/centos:dev
    860c279d2fec - 镜像id或镜像名
    runoob/centos:dev - 新标签名
  }
  top         显示容器的运行进程 | Display the running processes of a container
  unpause     取消暂停一个或多个容器中的所有进程 | Unpause all processes within one or more containers
  update      更新一个或多个容器的配置 | Update configuration of one or more containers
  version     显示Docker版本信息 | Show the Docker version information
  wait        阻塞，直到一个或多个容器停止，然后打印退出代码 | Block until one or more containers stop, then print their exit codes
}

Run 'docker COMMAND --help' for more information on a command.

常用操作命令{
  连接正在运行的容器: docker exec -it mariadb bash  // 这里连接"mariadb"并运行"bash"
  查看容器日志: docker logs mariadb // 这里查看"mariadb"容器的日志
  查看所有容器运行状态: docker stats  mariadb // 这里查看"mariadb"容器状态,缺省为查看所有容器
  docker查看运行容器详细信息: docker inspect  容器id/image
}
```



## 参考链接

[Docker 官网](http://www.docker.com) [Docker打包SpringBoot为镜像](https://www.jianshu.com/p/1d7a1bbd9317)

