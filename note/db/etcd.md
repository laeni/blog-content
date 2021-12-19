---

title: 'Etcd 帮助文档'
author: 'Laeni'
tags: 'db, etcd, k8s'
date: '2021-12-18'
updated: '2021-12-18'
---

本文档内容来自[官方3.5版本文档](https://etcd.io/docs/v3.5)。

## Help

### etcd

#### 使用 | Usage

```sh
etcd [flags]
  启动一个 etcd 服务器。| Start an etcd server.
etcd --version
  显示 etcd 版本。| Show the version of etcd.
etcd -h | --help
  显示有关 etcd 的帮助信息。| Show the help information about etcd.
etcd --config-file
  服务器配置文件的路径。| Path to the server configuration file.
  请注意，如果提供了配置文件，其他命令行标志和环境变量将被忽略。| Note that if a configuration file is provided, other command line flags and environment variables will be ignored.
etcd gateway
  运行无状态直通 etcd TCP 连接转发代理。| Run the stateless pass-through etcd TCP connection forwarding proxy.
etcd grpc-proxy
  运行无状态 etcd v3 gRPC L7 反向代理。| Run the stateless etcd v3 gRPC L7 reverse proxy.
```

#### 成员 | Member
```sh
--name 'default'
  此成员的可读名称。| Human-readable name for this member.
--data-dir '${name}.etcd'
  数据目录的路径。| Path to the data directory.
--wal-dir ''
  专用 wal 目录的路径。| Path to the dedicated wal directory.
--snapshot-count '100000'
  触发磁盘快照的已提交事务数。| Number of committed transactions to trigger a snapshot to disk.
--heartbeat-interval '100'
  心跳间隔的时间（以毫秒为单位）。| Time (in milliseconds) of a heartbeat interval.
--election-timeout '1000'
  选举超时的时间（以毫秒为单位）。 有关详细信息，请参阅调整文档。| Time (in milliseconds) for an election to timeout. See tuning documentation for details.
--initial-election-tick-advance 'true'
  是否在启动时快进初始选举滴答以加快选举速度。| Whether to fast-forward initial election ticks on boot for faster election.
--listen-peer-urls 'http://localhost:2380'
  用于侦听对等流量的 URL 列表。| List of URLs to listen on for peer traffic.
--listen-client-urls 'http://localhost:2379'
  用于侦听客户端流量的 URL 列表。| List of URLs to listen on for client traffic.
--max-snapshots '5'
  要保留的最大快照文件数（0 表示无限制）。| Maximum number of snapshot files to retain (0 is unlimited).
--max-wals '5'
  要保留的最大 wal 文件数（0 是无限制）。| Maximum number of wal files to retain (0 is unlimited).
--quota-backend-bytes '0'
  当后端大小超过给定的配额时发出警报（0 默认为低空间配额）。| Raise alarms when backend size exceeds the given quota (0 defaults to low space quota).
--backend-bbolt-freelist-type 'map'
  BackendFreelistType 指定boltdb 后端使用的freelist 的类型（array 和map 是支持的类型）。| BackendFreelistType specifies the type of freelist that boltdb backend uses(array and map are supported types).
--backend-batch-interval ''
  BackendBatchInterval 是提交后端事务之前的最长时间。| BackendBatchInterval is the maximum time before commit the backend transaction.
--backend-batch-limit '0'
  BackendBatchLimit 是提交后端事务之前的最大操作数。| BackendBatchLimit is the maximum operations before commit the backend transaction.
--max-txn-ops '128'
  事务中允许的最大操作数。| Maximum number of operations permitted in a transaction.
--max-request-bytes '1572864'
  服务器将接受的最大客户端请求大小（以字节为单位）。| Maximum client request size in bytes the server will accept.
--grpc-keepalive-min-time '5s'
  客户端在 ping 服务器之前应等待的最小持续时间间隔。| Minimum duration interval that a client should wait before pinging server.
--grpc-keepalive-interval '2h'
  服务器到客户端 ping 检查连接是否处于活动状态的频率持续时间（0 表示禁用）。| Frequency duration of server-to-client ping to check if a connection is alive (0 to disable).
--grpc-keepalive-timeout '20s'
  关闭无响应连接之前的额外等待时间（0 表示禁用）。| Additional duration of wait before closing a non-responsive connection (0 to disable).
--socket-reuse-port 'false'
  启用以在侦听器上设置套接字选项 SO_REUSEPORT，允许重新绑定已在使用的端口。| Enable to set socket option SO_REUSEPORT on listeners allowing rebinding of a port already in use.
--socket-reuse-address 'false'
  启用以在侦听器上设置套接字选项 SO_REUSEADDR，允许绑定到 TIME_WAIT 状态的地址。| Enable to set socket option SO_REUSEADDR on listeners allowing binding to an address in TIME_WAIT state.
```
#### 集群 | Clustering

```sh
--initial-advertise-peer-urls 'http://localhost:2380'
  此成员的对等 URL 列表，用于向集群的其余部分做广播。| List of this member's peer URLs to advertise to the rest of the cluster.
--initial-cluster 'default=http://localhost:2380'
  用于引导的初始集群配置。| Initial cluster configuration for bootstrapping.
--initial-cluster-state 'new'
  初始集群状态（“新-new”或“现有-existing”）。| Initial cluster state ('new' or 'existing').
--initial-cluster-token 'etcd-cluster'
  引导期间 etcd 集群的初始集群令牌。| Initial cluster token for the etcd cluster during bootstrap.
  指定此项可以防止您在运行多个集群时发生意外的跨集群交互。Specifying this can protect you from unintended cross-cluster interaction when running multiple clusters.
--advertise-client-urls 'http://localhost:2379'
  此成员要向公众发布广播的客户端 URL 的列表。| List of this member's client URLs to advertise to the public.
  与 etcd 集群通信的机器应该可以访问广播的客户端 URL。 etcd 客户端库解析这些 URL 以连接到集群。| The client URLs advertised should be accessible to machines that talk to etcd cluster. etcd client libraries parse these URLs to connect to the cluster.
--discovery ''
  用于引导集群的发现 URL。| Discovery URL used to bootstrap the cluster.
--discovery-fallback 'proxy'
  发现服务失败时的预期行为（“退出-exit”或“代理-proxy”）。| Expected behavior ('exit' or 'proxy') when discovery services fails.
  “代理”仅支持 v2 API。| "proxy" supports v2 API only.
--discovery-proxy ''
  用于发现服务的流量的 HTTP 代理。| HTTP proxy to use for traffic to discovery service.
--discovery-srv ''
  用于引导集群的 DNS srv 域。| DNS srv domain used to bootstrap the cluster.
--discovery-srv-name ''
  引导时查询的 dns srv 名称的后缀。| Suffix to the dns srv name queried when bootstrapping.
--strict-reconfig-check 'true'
  拒绝会导致仲裁丢失的重新配置请求。| Reject reconfiguration requests that would cause quorum loss.
--pre-vote 'true'
  启用以运行额外的 Raft 选举阶段。| Enable to run an additional Raft election phase.
--auto-compaction-retention '0'
  自动压缩长度。0 表示禁用自动压缩。| Auto compaction retention length. 0 means disable auto compaction.
--auto-compaction-mode 'periodic'
  解释“自动压缩保留”之一：定期-periodic|修订-revision。 'periodic' 用于基于持续时间的保留，如果未提供时间单位则默认为小时（例如 '5m'）。 'revision' 用于基于修订号的保留。| Interpret 'auto-compaction-retention' one of: periodic|revision. 'periodic' for duration based retention, defaulting to hours if no time unit is provided (e.g. '5m'). 'revision' for revision number based retention.
--enable-v2 'false'
  接受 etcd V2 客户端请求。已弃用并将在 v3.6 中停用。| Accept etcd V2 client requests. Deprecated and to be decommissioned in v3.6.
--v2-deprecation 'not-yet'
  v2store 弃用阶段。允许选择更高的兼容性模式。| Phase of v2store deprecation. Allows to opt-in for higher compatibility mode.
  Supported values:
    'not-yet'                // 如果 v2store 具有有意义的内容，则发出警告（v3.5 中的默认值）| Issues a warning if v2store have meaningful content (default in v3.5)
    'write-only'             // 不允许自定义 v2 状态（v3.6 中的计划默认值）| Custom v2 state is not allowed (planned default in v3.6)
    'write-only-drop-data'   // 自定义 v2 状态将被删除！| Custom v2 state will get DELETED !
    'gone'                   // v2store 不再维护。 （v3.7 中的计划默认值）| v2store is not maintained any longer. (planned default in v3.7)
```

#### 安全 | Security

```sh
--cert-file ''
  客户端服务器 TLS 证书文件的路径。| Path to the client server TLS cert file.
--key-file ''
  客户端服务器 TLS 密钥文件的路径。| Path to the client server TLS key file.
--client-cert-auth 'false'
  启用客户端证书身份验证。| Enable client cert authentication.
--client-crl-file ''
  客户端证书吊销列表文件的路径。| Path to the client certificate revocation list file.
--client-cert-allowed-hostname ''
  允许用于客户端证书身份验证的 TLS 主机名。| Allowed TLS hostname for client cert authentication.
--trusted-ca-file ''
  客户端服务器 TLS 可信 CA 证书文件的路径。| Path to the client server TLS trusted CA cert file.
--auto-tls 'false'
  使用生成的证书的客户端 TLS。| Client TLS using generated certificates.
--peer-cert-file ''
  对等服务器 TLS 证书文件的路径。| Path to the peer server TLS cert file.
--peer-key-file ''
  对等服务器 TLS 密钥文件的路径。| Path to the peer server TLS key file.
--peer-client-cert-auth 'false'
  启用对等客户端证书身份验证。| Enable peer client cert authentication.
--peer-trusted-ca-file ''
  对等服务器 TLS 可信 CA 文件的路径。| Path to the peer server TLS trusted CA file.
--peer-cert-allowed-cn ''
  连接到对等端点的客户端证书所需的 CN。| Required CN for client certs connecting to the peer endpoint.
--peer-cert-allowed-hostname ''
  允许使用 TLS 主机名进行对等身份验证。| Allowed TLS hostname for inter peer authentication.
--peer-auto-tls 'false'
  如果未提供 --peer-key-file 和 --peer-cert-file ，则使用自生成证书的对等 TLS。| Peer TLS using self-generated certificates if --peer-key-file and --peer-cert-file are not provided.
--self-signed-cert-validity '1'
  指定 ClientAutoTLS 和 PeerAutoTLS 时 etcd 自动生成的 client 和 peer 证书的有效期，单位为 year，默认为1。| The validity period of the client and peer certificates that are automatically generated by etcd when you specify ClientAutoTLS and PeerAutoTLS, the unit is year, and the default is 1.
--peer-crl-file ''
  对等证书吊销列表文件的路径。| Path to the peer certificate revocation list file.
--cipher-suites ''
  客户端/服务器和对等点之间支持的 TLS 密码套件的逗号分隔列表（空将由 Go 自动填充）。| Comma-separated list of supported TLS cipher suites between client/server and peers (empty will be auto-populated by Go).
--cors '*'
  逗号分隔的 CORS 源白名单，或跨源资源共享，（空或 * 表示允许全部）。| Comma-separated whitelist of origins for CORS, or cross-origin resource sharing, (empty or * means allow all).
--host-whitelist '*'
  来自 HTTP 客户端请求的可接受主机名，如果服务器不安全（空或 * 表示允许全部）。| Acceptable hostnames from HTTP client requests, if server is not secure (empty or * means allow all).
```

#### 认证 | Auth

```sh
--auth-token 'simple'
  指定 v3 身份验证令牌类型及其选项（“简单-simple”或“jwt”）。| Specify a v3 authentication token type and its options ('simple' or 'jwt').
--bcrypt-cost 10
  指定用于散列身份验证密码的 bcrypt 算法的成本/强度。 有效值介于 4 和 31 之间。| Specify the cost / strength of the bcrypt algorithm for hashing auth passwords. Valid values are between 4 and 31.
--auth-token-ttl 300
  auth-token-ttl 的时间（以秒为单位）。| Time (in seconds) of the auth-token-ttl.
```

#### 分析和监控| Profiling and Monitoring

```sh
--enable-pprof 'false'
  Enable runtime profiling data via HTTP server. Address is at client URL + "/debug/pprof/"
--metrics 'basic'
  Set level of detail for exported metrics, specify 'extensive' to include server side grpc histogram metrics.
--listen-metrics-urls ''
  List of URLs to listen on for the metrics and health endpoints. 
```

#### 日志 | Logging

```sh
--logger 'zap'
  目前仅支持结构化日志记录的“zap”。| Currently only supports 'zap' for structured logging.
--log-outputs 'default'
  指定 'stdout' 或 'stderr' 以跳过日志记录，即使在 systemd 或逗号分隔的输出目标列表下运行时也是如此。| Specify 'stdout' or 'stderr' to skip journald logging even when running under systemd, or list of comma separated output targets.
--log-level 'info'
  配置日志级别。| Configures log level. Only supports debug, info, warn, error, panic, or fatal.
--enable-log-rotation 'false'
  启用单个日志输出文件目标的日志轮换。| Enable log rotation of a single log-outputs file target.
--log-rotation-config-json '{"maxsize": 100, "maxage": 0, "maxbackups": 0, "localtime": false, "compress": false}'
  如果使用 JSON 记录器配置启用，则配置日志轮换。 MaxSize(MB), MaxAge(days,0=no limit), MaxBackups(0=no limit), LocalTime(使用计算机本地时间), Compress(gzip)。| Configures log rotation if enabled with a JSON logger config. MaxSize(MB), MaxAge(days,0=no limit), MaxBackups(0=no limit), LocalTime(use computers local time), Compress(gzip). 
```

#### 实验性分布式追踪 | Experimental distributed tracing

```sh
--experimental-enable-distributed-tracing 'false'
  Enable experimental distributed tracing.
--experimental-distributed-tracing-address 'localhost:4317'
  Distributed tracing collector address.
--experimental-distributed-tracing-service-name 'etcd'
  Distributed tracing service name, must be same across all etcd instances.
--experimental-distributed-tracing-instance-id ''
  Distributed tracing instance ID, must be unique per each etcd instance.
```

#### v2 代理（将在 v3.6 中弃用） | v2 Proxy (to be deprecated in v3.6)

```sh
--proxy 'off'
  Proxy mode setting ('off', 'readonly' or 'on').
--proxy-failure-wait 5000
  Time (in milliseconds) an endpoint will be held in a failed state.
--proxy-refresh-interval 30000
  Time (in milliseconds) of the endpoints refresh interval.
--proxy-dial-timeout 1000
  Time (in milliseconds) for a dial to timeout.
--proxy-write-timeout 5000
  Time (in milliseconds) for a write to timeout.
--proxy-read-timeout 0
  Time (in milliseconds) for a read to timeout.
```

#### 实验功能 | Experimental feature

```sh
--experimental-initial-corrupt-check 'false'
  Enable to check data corruption before serving any client/peer traffic.
--experimental-corrupt-check-time '0s'
  Duration of time between cluster corruption check passes.
--experimental-enable-v2v3 ''
  Serve v2 requests through the v3 backend under a given prefix. Deprecated and to be decommissioned in v3.6.
--experimental-enable-lease-checkpoint 'false'
  ExperimentalEnableLeaseCheckpoint enables primary lessor to persist lease remainingTTL to prevent indefinite auto-renewal of long lived leases.
--experimental-compaction-batch-limit 1000
  ExperimentalCompactionBatchLimit sets the maximum revisions deleted in each compaction batch.
--experimental-peer-skip-client-san-verification 'false'
  Skip verification of SAN field in client certificate for peer connections.
--experimental-watch-progress-notify-interval '10m'
  Duration of periodical watch progress notification.
--experimental-warning-apply-duration '100ms'
      Warning is generated if requests take more than this duration.
--experimental-txn-mode-write-with-shared-buffer 'true'
  Enable the write transaction to use a shared buffer in its readonly check operations.
--experimental-bootstrap-defrag-threshold-megabytes
  Enable the defrag during etcd server bootstrap on condition that it will free at least the provided threshold of disk space. Needs to be set to non-zero value to take effect.
```

#### 不安全功能 | Unsafe feature

> 小心不安全的标志！ 它可能会破坏共识协议给出的保证！| CAUTIOUS with unsafe flag! It may break the guarantees given by the consensus protocol!

```sh
--force-new-cluster 'false'
  Force to create a new one-member cluster.
--unsafe-no-fsync 'false'
  Disables fsync, unsafe, will cause data loss.
```

### etcdctl

etcdctl 使用实例请参考[示例](#示例)

## 安装

### 集群的“多数”与“容错”

多数(Majority)：多数是为了仲裁时投票表决的成员数。值为：(节点数 / 2) + 1

容错(Failure Tolerance)：容错时集群内允许不可用（如宕机）节点的数量。

以下为集群节点数与“多数”和“容错”的关系。

| 集群节点数 | 多数 | 容错 |
| :--------: | :--: | :--: |
|     1      |  1   |  0   |
|     2      |  2   |  0   |
|     3      |  2   |  1   |
|     4      |  3   |  1   |
|     5      |  3   |  2   |
|     6      |  4   |  2   |
|     7      |  4   |  3   |
|     8      |  5   |  3   |
|     9      |  5   |  4   |

### 单机安装（官方Docker示例）

```sh
# 初始化数据挂在路径
$ sudo rm -rf /tmp/etcd-data.tmp && mkdir -p /tmp/etcd-data.tmp
# 启动
$ docker run \
  --rm \
  -p 2379:2379 \
  -p 2380:2380 \
  -v /etc/localtime:/etc/localtime \
  --mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
  --name etcd-gcr-v3.5.1 \
  quay.io/coreos/etcd:v3.5.1 \
  /usr/local/bin/etcd \
  --name s1 \
  --data-dir /etcd-data \
  --listen-client-urls http://0.0.0.0:2379 \ # 用于侦听客户端流量的 URL 列表
  --listen-peer-urls http://0.0.0.0:2380 \ # 用于侦听对等流量的 URL 列表
  --advertise-client-urls http://0.0.0.0:2379 \ # 此成员要向公众发布广播的客户端 URL 的列表
  --initial-advertise-peer-urls http://0.0.0.0:2380 \ # 此成员的对等 URL 列表，用于向集群的其余部分做广播。
  --initial-cluster s1=http://0.0.0.0:2380 \ # 这里的‘s1‘为‘--name’选项的值，即可以自定义
  --initial-cluster-token tkn \ # 该值可以自定义
  --initial-cluster-state new \
  --log-level info \
  --logger zap \
  --log-outputs stderr
```

### 集群安装

#### 集群成员信息（规划）

> ```shell
> TOKEN=tkn # 引导期间 etcd 集群的初始集群令牌。| 任意定义
> CLUSTER_STATE=new # 初始集群状态（“新-new”或“现有-existing”）
> NAME_1=machine-1 # 节点1名称
> NAME_2=machine-2 # 节点2名称
> NAME_3=machine-3 # 节点3名称
> HOST_1=10.240.0.17 # 节点1IP
> HOST_2=10.240.0.18 # 节点2IP
> HOST_3=10.240.0.19 # 节点3IP
> CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380
> ```

#### 使用Docker创建一个etcd专用网络

```sh
$ docker network create \
  --gateway 10.240.0.1 \
  --subnet 10.240.0.0/24 \
  etcd-net
```

#### 使用Docker依次启动三个实例

```sh
# For machine 1
THIS_NAME=${NAME_1}
THIS_IP=${HOST_1}
ETCD_DATA=/tmp/etcd-data.tmp.1

# For machine 2
THIS_NAME=${NAME_2}
THIS_IP=${HOST_2}
ETCD_DATA=/tmp/etcd-data.tmp.2

# For machine 3
THIS_NAME=${NAME_3}
THIS_IP=${HOST_3}
ETCD_DATA=/tmp/etcd-data.tmp.3

# 上面三段，依次在单独的窗口执行完一段后就执行这一段创建一个实例(由于仅作为演示，用完即删，所以创建时加上 --rm 选项)
$ sudo rm -rf ${ETCD_DATA} && mkdir -p ${ETCD_DATA} && \
  docker run --rm \
  --name edct_${THIS_NAME} \
  --net etcd-net \
  --ip ${THIS_IP} \
  -v /etc/localtime:/etc/localtime \
  --mount type=bind,source=${ETCD_DATA},destination=/etcd-data \
  quay.io/coreos/etcd:v3.5.1 \
  /usr/local/bin/etcd \
    --name ${THIS_NAME} \
    --data-dir /etcd-data \
    --initial-advertise-peer-urls http://${THIS_IP}:2380 \
    --listen-peer-urls http://${THIS_IP}:2380 \
    --advertise-client-urls http://${THIS_IP}:2379 \
    --listen-client-urls http://${THIS_IP}:2379 \
    --initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} \
	--initial-cluster-token ${TOKEN} \
    --log-level info \
    --logger zap \
    --log-outputs stderr

# 【备案】集群引导时，除了明确指定集群地址外，还可以使用发现协议进行发现，但是需要额外部署一个发现服务（一般使用公共发现服务）
## 详情参考：https://etcd.io/docs/v3.5/dev-internal/discovery_protocol/
$ curl https://discovery.etcd.io/new?size=3
https://discovery.etcd.io/a81b5818e67a6ea83e9d4daea5ecbc92 # a81b5818e67a6ea83e9d4daea5ecbc92 为本地生成的随机值
$ etcd \
    ... \
	# 替换 --initial-cluster ${CLUSTER} 为发现地址
	--discovery https://discovery.etcd.io/a81b5818e67a6ea83e9d4daea5ecbc92 \
	...
```

#### 验证集群

```sh
export ETCDCTL_API=3
HOST_1=10.240.0.17
HOST_2=10.240.0.18
HOST_3=10.240.0.19
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379

# 列出集群列表（如果此时能看到刚刚添加的三个节点且所有节点日志输出正常则说明集群搭建成功）
etcdctl --endpoints=$ENDPOINTS member list
```

## 示例（etcdctl）

### 常用操作

```sh
# 写入数据
etcdctl --endpoints=$ENDPOINTS put foo xxx
etcdctl --endpoints=$ENDPOINTS put foo1 xxx1
etcdctl --endpoints=$ENDPOINTS put foo2 xxx2
# 获取写入的数据
etcdctl --endpoints=$ENDPOINTS get foo
etcdctl --endpoints=$ENDPOINTS --write-out="json" get foo
# 获取相同前缀的key
etcdctl --endpoints=$ENDPOINTS get foo --prefix
# 删除key
etcdctl --endpoints=$ENDPOINTS del foo
etcdctl --endpoints=$ENDPOINTS del foo --prefix
```

### 事务

```sh
# 在事务中进行多次写入(txn 将多个请求包装到一个事务中)
etcdctl --endpoints=$ENDPOINTS put user1 bad # 这里先put一个值，用于事物的条件
etcdctl --endpoints=$ENDPOINTS txn --interactive
compares: # 这里写本次事物的条件（每行为一个语句，遇到空语句则表示输入完成）
value("user1") = "bad" # 输入

success requests (get, put, delete): # 这里写条件匹配时执行的语句（每行为一个语句，遇到空语句则表示输入完成）
del user1
put xxx ABCDEF

failure requests (get, put, delete): # 这里写条件不匹配时执行的语句（每行为一个语句，遇到空语句则表示输入完成）
put user1 good
```

### 监听

```sh
# 在会话1监听“stock1”
etcdctl --endpoints=$ENDPOINTS watch stock1
# 在会话2操作“stock1”
etcdctl --endpoints=$ENDPOINTS put stock1 1000
etcdctl --endpoints=$ENDPOINTS del stock1

# 在会话1监听“stock”前缀的所有值
etcdctl --endpoints=$ENDPOINTS watch stock --prefix
# 在会话2监听写入“stock”前缀的key
etcdctl --endpoints=$ENDPOINTS put stock1 10
etcdctl --endpoints=$ENDPOINTS put stock2 20
```

### 租约（有效期）

```sh
# 创建一个租约（有效期为120秒），一旦创建成功后就开始计时，需要在该租约的有效期内使用它
$ etcdctl --endpoints=$ENDPOINTS lease grant 120
lease 2be7547fbc6a5afa granted with TTL(120s)
# put 值时指定租约（如果租约过期则操作失败）
$ etcdctl --endpoints=$ENDPOINTS put sample value --lease=2be7547fbc6a5afa
# 如果在有效期外获取该值时会返回空
$ etcdctl --endpoints=$ENDPOINTS get sample
# 续租（需要注意续租规则）
$ etcdctl --endpoints=$ENDPOINTS lease keep-alive 2be7547fbc6a5afa
# 立即删除该租约
$ etcdctl --endpoints=$ENDPOINTS lease revoke 2be7547fbc6a5afa
```

### 分布式锁

```sh
# 会话1使用锁‘mutex1’
$ etcdctl --endpoints=$ENDPOINTS lock mutex1
mutex1/15d57dd2841a9748

# 另一个会话使用相同名称的锁时会阻塞，直到会话1释放该锁时才会继续获得锁
$ etcdctl --endpoints=$ENDPOINTS lock mutex1
```

### leader选举

```sh
# 和锁类似，会话1（p1）先执行时它会当选为 leader
$ etcdctl --endpoints=$ENDPOINTS elect one p1
one/37197dd283fe529e
p1

# 另一个会话（p2）也参与选举时时会阻塞，直到会话1“终止”时p1才会当选
$ etcdctl --endpoints=$ENDPOINTS elect one p2
```

## 管理

### 查看集群状态

```sh
# 以表格形式输出集群状态
$ etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status
+------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 10.240.0.17:2379 | 4917a7ab173fabe7 |  3.5.0  |   45 kB |      true |      false |         4 |      16726 |              16726 |        |
| 10.240.0.18:2379 | 59796ba9cd1bcd72 |  3.5.0  |   45 kB |     false |      false |         4 |      16726 |              16726 |        |
| 10.240.0.19:2379 | 94df724b66343e6c |  3.5.0  |   45 kB |     false |      false |         4 |      16726 |              16726 |        |
+------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------|

# 检查集群是否健康
$ etcdctl --endpoints=$ENDPOINTS endpoint health
10.240.0.17:2379 is healthy: successfully committed proposal: took = 3.345431ms
10.240.0.19:2379 is healthy: successfully committed proposal: took = 3.767967ms
10.240.0.18:2379 is healthy: successfully committed proposal: took = 4.025451ms
```

### 创建快照（备份）

```sh
# 这里的 endpoints 不能使用集群地址，而是只能使用单个节点的地址，以明确表示备份哪一个节点
$ etcdctl --endpoints=10.240.0.17:2379 snapshot save my.db
```

### 添加和删除集群成员

本步骤是在“集群安装”的基础上进行。

```sh
# step.1 获取成员ID | get member ID
$ echo $ENDPOINTS
10.240.0.17:2379,10.240.0.18:2379,10.240.0.19:2379
$ etcdctl --endpoints=$ENDPOINTS member list  --write-out=table
+------------------+---------+-----------+-------------------------+-------------------------+------------+
|        ID        | STATUS  |   NAME    |       PEER ADDRS        |      CLIENT ADDRS       | IS LEARNER |
+------------------+---------+-----------+-------------------------+-------------------------+------------+
| 373c3e8632453c35 | started | machine-3 | http://10.240.0.19:2380 | http://10.240.0.19:2379 |      false |
| 725017df920e15d5 | started | machine-2 | http://10.240.0.18:2380 | http://10.240.0.18:2379 |      false |
| e488564abad0b719 | started | machine-1 | http://10.240.0.17:2380 | http://10.240.0.17:2379 |      false |
+------------------+---------+-----------+-------------------------+-------------------------+------------+

# step.2 删除ID为‘373c3e8632453c35’的成员 | remove the member
MEMBER_ID=373c3e8632453c35
$ etcdctl --endpoints=$ENDPOINTS member remove ${MEMBER_ID}

# step.3 添加一个新成员（此时新成员还没启动） | add a new member (node 4)
export ETCDCTL_API=3
NAME_1=machine-1
NAME_2=machine-2
NAME_4=machine-4
HOST_1=10.240.0.17
HOST_2=10.240.0.18
HOST_4=10.240.0.20 # new member
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_4:2379
etcdctl --endpoints=${HOST_1}:2379,${HOST_2}:2379 \
	member add ${NAME_4} \
	--peer-urls=http://${HOST_4}:2380

# step.4 使用--initial-cluster-state existing标志启动新成员
TOKEN=tkn # 引导期间 etcd 集群的初始集群令牌。| 任意定义
CLUSTER_STATE=existing # 初始集群状态（“新-new”或“现有-existing”）
NAME_1=machine-1 # 节点1名称
NAME_2=machine-2 # 节点2名称
NAME_4=machine-4 # 节点4名称(新)
HOST_1=10.240.0.17 # 节点1IP
HOST_2=10.240.0.18 # 节点2IP
HOST_4=10.240.0.20 # 节点4IP(新)
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_4}=http://${HOST_4}:2380
# For machine 4(新)
THIS_NAME=${NAME_4}
THIS_IP=${HOST_4}
ETCD_DATA=/tmp/etcd-data.tmp.4
# 该行命令和启动前面三个成员的命令完全相同
$ sudo rm -rf ${ETCD_DATA} && mkdir -p ${ETCD_DATA} && \
  docker run --rm \
  --name edct_${THIS_NAME} \
  --net etcd-net \
  --ip ${THIS_IP} \
  -v /etc/localtime:/etc/localtime \
  --mount type=bind,source=${ETCD_DATA},destination=/etcd-data \
  quay.io/coreos/etcd:v3.5.1 \
  /usr/local/bin/etcd \
    --name ${THIS_NAME} \
    --data-dir /etcd-data \
    --initial-advertise-peer-urls http://${THIS_IP}:2380 \
    --listen-peer-urls http://${THIS_IP}:2380 \
    --advertise-client-urls http://${THIS_IP}:2379 \
    --listen-client-urls http://${THIS_IP}:2379 \
    --initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} \
	--initial-cluster-token ${TOKEN} \
    --log-level info \
    --logger zap \
    --log-outputs stderr
```



## [认证](https://etcd.io/docs/v3.5/demo/#auth)

`auth`, `user`,`role`用于身份验证

```sh
export ETCDCTL_API=3
ENDPOINTS=localhost:2379

etcdctl --endpoints=${ENDPOINTS} role add root
etcdctl --endpoints=${ENDPOINTS} role get root

etcdctl --endpoints=${ENDPOINTS} user add root
etcdctl --endpoints=${ENDPOINTS} user grant-role root root
etcdctl --endpoints=${ENDPOINTS} user get root

etcdctl --endpoints=${ENDPOINTS} role add role0
etcdctl --endpoints=${ENDPOINTS} role grant-permission role0 readwrite foo
etcdctl --endpoints=${ENDPOINTS} user add user0
etcdctl --endpoints=${ENDPOINTS} user grant-role user0 role0

etcdctl --endpoints=${ENDPOINTS} auth enable
# now all client requests go through auth

etcdctl --endpoints=${ENDPOINTS} --user=user0:123 put foo bar
etcdctl --endpoints=${ENDPOINTS} get foo
# permission denied, user name is empty because the request does not issue an authentication request
etcdctl --endpoints=${ENDPOINTS} --user=user0:123 get foo
# user0 can read the key foo
etcdctl --endpoints=${ENDPOINTS} --user=user0:123 get foo1
```

## 其他

### 工具

<https://etcdmanager.io/>

### 备份 etcd 集群（快照）

etcdctl 提供了`snapshot`创建备份的命令。有关详细信息，请参阅[备份](https://etcd.io/docs/v3.5/op-guide/recovery/#snapshotting-the-keyspace)。



## 注意事项

1. 替换 etcd 集群节点时，重要的是先删除成员，然后添加其替换。有关详细信息，请参阅[官方文档](https://etcd.io/docs/v3.5/faq/#should-i-add-a-member-before-removing-an-unhealthy-member)。

## 高级文档

- [学习](https://etcd.io/docs/v3.5/learning/)
- [操作指南](https://etcd.io/docs/v3.5/op-guide/)

