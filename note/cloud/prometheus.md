---
title: Prometheus 快速入门
tags: Prometheus
author: Laeni
date: 2024-07-03
updated: 2024-11-22
---

![image-20240703111929125](./prometheus.assets/image-20240703111929125.png)

# 服务端（Server）

## 安装

```sh
cat > prometheus.yml <<EOF
# my global config
global:
  scrape_interval: 15s # 将抓取间隔设置为每 15 秒。默认为每 1 分钟。
  evaluation_interval: 15s # 每 15 秒评估一次规则。默认值为每 1 分钟。
  # scrape_timeout 设置为全局默认值 (10s)。

  # 与外部系统（联合、远程存储、Alertmanager）通信时将这些标签附加到任何时间序列或警报。
  external_labels:
    monitor: 'codelab-monitor'

# Alertmanager 配置
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# 加载规则一次并根据全局“evaluation_interval”定期对其进行评估。
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# 一个抓取配置只包含一个要抓取的端点
scrape_configs:
  # 里是 Prometheus 本身。
  # 作业名称作为标签`job=<job_name>`添加到从此配置中抓取的任何时间序列。
  - job_name: "prometheus"

    # metrics_path 默认为 '/metrics'
    # scheme 默认为 'http'.

    static_configs:
      - targets: ["localhost:9090"]
EOF

PWD=$(dirname $(readlink -f "$0"))

docker run -d \
  --restart=always \
  --name prometheus \
  -p 9090:9090 \
  -v /etc/localtime:/etc/localtime \
  -v ${PWD}/prometheus:/prometheus `# 数据存储目录` \
  -v ${PWD}/prometheus.yml:/etc/prometheus/prometheus.yml `# 配置文件` \
  prom/prometheus:v2.45.6
```

上述命令将使用默认配置（部分配置来源于官网[开始使用](https://prometheus.io/docs/prometheus/latest/getting_started/)）启动服务端，并将自己加入监控。

## 使用

安装后可通过<http://localhost:9090/graph>进入控制台。

有关表达式语言的详细信息，请参阅[表达式语言文档](https://prometheus.io/docs/prometheus/latest/querying/basics/)。

## 发现

Prometheus 提供了多种[服务发现选项](https://github.com/prometheus/prometheus/tree/main/discovery)来发现抓取目标，包括 [Kubernetes](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)、[Consul](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#consul_sd_config) 和[基于文件的服务发现](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config)等。

### 示例

#### 基于文件的服务发现

**prometheus.yml**：

```yaml
scrape_configs:
- job_name: 'node'
  file_sd_configs:
  - files:
    - 'targets.json'
```

**targets.json**：

```json
[
  { "targets": ["localhost:9100"], "labels": { "job": "node" } },
  { "targets": ["localhost:9101"], "labels": { "job": "node" } }
]
```

#### 在 Spring Actuator 中添加 prometheus 端点提供指标服务

##### Spring Boot 配置

**pom.xml**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

**application.yml**：

```yaml
server:
  port: 8080
  servlet:
    context-path: /project-name
management:
  endpoints:
    web:
      exposure:
        include:
          - prometheus # 公开 Prometheus 指标
  metrics:
    tags:
      application: ${spring.application.name}
```

> 现在可以通过`/project-name/actuator/prometheus`访问公开的指标。

**统计 Controller**：

```java
@RestController
@io.micrometer.core.annotation.Timed(value = "http_api_request_duration", description = "接口响应时间", histogram = true)
public class Controller {}
```

**手动统计*Counter* 指标**：

```java
public class MyClass {

    private final Counter counter;

    public MyClass(MeterRegistry registry) {
        // 创建指标实例
        this.counter = Counter
                .builder("http_api_request_count")
                .description("HTTP 接口总请求次数")
                .tags("tag-1", "value-1", "tag-2", "value-2")
                .register(registry);
    }

    public void run() {
        // 增加指标计数
        this.counter.increment();
    }
}
```

##### Prometheus 配置

由于指标路径不是常见的 `/metrics` 格式，而是 `/project-name/actuator/prometheus` 格式（前缀根据项目的不同而不同），所以指标路径需要动态生成。

假设自动发现文件为 **sd-node.json**:

```json
[
  {
    "targets": ["10.10.1.3:8081"],
    "labels": { "project":"my-project" }
  }
]
```

即根据`project`标签来标识项目名，则 **prometheus.yml** 中关于自动发现部分的配置为：

```yaml
scrape_configs:
  - job_name: spring-actuators
    # 由于抓取路径随项目变化而变化，所以动态生成（使用自动发现文件配置的属性来生成）
    relabel_configs:
      # 此配置的含义: 使用正则匹配输入数据（'source_labels: [project]'），匹配后按'replacement'的规则进行替换，并将替换后的内容写入'__metrics_path__'标签
      # 1. `__metrics_path__`为实际指标抓取是使用的路径（不含主机部分）；2. `project` 为自动发现文件`conf.d/sd-ibox-spring-dev.json` 中配置的自定义标签；3. 所有可用的标签可以通过`/service-discovery`页面查看
      - source_labels: [project]
        regex: '(.*)'
        target_label: __metrics_path__
        replacement: '/$1/actuator/prometheus'
    # 使用文件来发现抓取目标 - https://prometheus.io/docs/guides/file-sd/
    file_sd_configs:
      - files: ['sd-node.json']
        refresh_interval: 30s
```

> 如果没有 `context-path`，则直接通过 `metrics_path: /actuator/prometheus` 配置即可。

# OpenMetrics

OpenMetrics，是云原生、高度可扩展的指标协议。 OpenMetrics 定义了大规模上报云原生指标的事实标准，并支持文本表示协议和 Protocol Buffers 协议。

## Micrometer指标类型

| **Micrometer指标类型** | **典型用途**                                                 |
| ---------------------- | ------------------------------------------------------------ |
| Counter                | 计数器，单调递增场景。例如，统计 PV 和 UV，接口调用次数等。  |
| Gauge                  | 持续波动的变量。例如，资源使用率、系统负载、请求队列长度等。 |
| Timer                  | 统计数据分布。例如，统计某接口调用延时的 P50、P90、P99 等。<br />有的实现叫做 *Histogram*。 |
| DistributionSummary    | 统计数据分布，与 Histogram 用途类似。有的实现叫做 *Summary*。 |

# 相关文档

- [Spring Boot应用如何快速接入Prometheus监控](https://help.aliyun.com/zh/prometheus/use-cases/connect-spring-boot-applications-to-managed-service-for-prometheus)

