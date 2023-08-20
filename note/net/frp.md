# frpc

## 安装

### OpenWRT

#### 软件包安装

1. 打开`系统-Software`软件页面。

2. 点击`Update lists...`更新软件源。

3. 在`Filter`输入框输入`frp`等过滤关键字。

4. 找到`luci-i18n-frpc-zh-cn`包安装即可。

   > 由于依赖关系，安装`luci-i18n-frpc-zh-cn`的同时会自动安装`luci-i18n-frpc-zh-cn`、`luci-app-frpc`和`frpc`三个包。

#### 配置

1. 进入`服务-frp Client`配置页面。

2. 根据需要在输入框填写与服务端通信的配置信息。

   ```yaml
   Server address: frp.exmple.com # frps 服务器主机地址
   Server port: 7000 # frps 监听端口
   TCP mux: true # 是否启用TCP连接多路复用（需要服务端也开启才生效）
   协议: kcp # 指定与服务器交互时使用的协议。有效值为'tcp'、'kcp'和'websocket'。默认值: 'tcp'。
   Additional settings: # 该列表可用于指定一些未包含在此 LuCI 中的附加参数。详情参考 frp 官方文档
       - pool_count: 2
       - authentication_method: token
       - token = <token>
   ```

3. 添加代理。

   1. 左下角填写**代理名称**后点击**添加**。

   2. 代理配置参考如下配置。

      ```yaml
      Proxy type: stcp # 需要代理的协议，目前页面不能添加 sudp，如需添加 sudp，可以先添加 stcp，然后到生成的配置中直接修改配置文件
      加密: true # 加密和Compression可以用于应对一些防火墙根据包特征拦截的情况
      Compression: true
      Local IP: 0.0.0.0 # 允许访问的来源IP，可以根据安全情况修改
      Local port: 51820 # 代理映射的本地端口
      ```

   > 可以直接修改配置文件`/etc/config/frpc`后重启。该配置文件应该为`luci-app-frpc`的配置文件，生效后生成临时**frpc**配置文件到`/tmp/etc/frpc.ini`。