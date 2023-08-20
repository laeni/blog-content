---
title: 'Maven pom.xml 常见配置'
author: 'Laeni'
tags: 'maven, pom'
date: '2023-08-06'
updated: '2023-08-06'
---

`pom.xml`的完整配置参见[XSD声明](https://maven.apache.org/xsd/maven-4.0.0.xsd)。

# 常用配置

## 环境配置

`profiles`配置可以指定一系列附加配置，这些配置可以在构建时明确指定激活（通过`-P a[,b]`参数）或者在某些条件下自动激活，一般多环境时使用。一旦配置被激活，则激活的配置将和其他配置合并。可以覆盖的配置有`build`、`modules`、`distributionManagement`、`properties`、`dependencyManagement`、`dependencies`、`repositories`、`pluginRepositories`、`reports`、`reporting`。

### 示例

```xml
<project>
    <profiles>
        <profile>
            <!-- 唯一标识一个 profile -->
            <id>dev</id>
            <!-- 激活条件 -->
            <activation>
            	<!-- 是否默认激活。当指定 -P 或者此文件中的其他 profile 被激活时 activeByDefault 失效 -->
                <activeByDefault>true</activeByDefault>
            </activation>
            ...... 其他需要合并的配置 ......
        </profile>
        <profile>
            <id>prd</id>
            ...... 其他需要合并的配置 ......
        </profile>
    </profiles>
</project>
```

