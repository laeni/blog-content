---
title: Prometheus 快速入门
tags: Prometheus
author: Laeni
date: 2024-07-03
updated: 2024-07-03
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

