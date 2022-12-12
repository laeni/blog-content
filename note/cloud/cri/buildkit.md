[buildkit](https://github.com/moby/buildkit) 项目是 Docker 公司开源出来的一个构建工具包，支持 OCI 标准的镜像构建。它主要包含以下部分:

- 服务端 buildkitd，当前支持 runc 和 containerd 作为 worker，默认是 runc；
- 客户端 buildctl，负责解析 Dockerfile，并向服务端 buildkitd 发出构建请求。