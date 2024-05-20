# Pentaho 套件

## 服务器

### Pentaho 服务器



## 客户端工具

### Pentaho Aggregation Designer - 聚合设计器

### Pentaho Data Integration - 数据集成

#### 安装

从[官网](https://www.hitachivantara.com/en-us/products/pentaho-plus-platform/data-integration-analytics/download-pentaho.html)下载社区版（当前最新版本：`pdi-ce-9.4.0.0-343.zip`）。

#### [命令行工具](https://help.hitachivantara.com/Documentation/Pentaho/Data_Integration_and_Analytics/9.4/Products/Use_Command_Line_Tools_to_Run_Transformations_and_Jobs)

- Pan 是 PDI 用于 执行转换的命令行工具。它可以执行本地或远程存储卡的转换。

  用法: `pan.sh -option=value arg1 arg2`

  | Switch        | Purpose                                                      |
  | ------------- | ------------------------------------------------------------ |
  | rep           | 企业或数据库存储库名称（如果使用）                           |
  | user          | 存储库用户名                                                 |
  | pass          | 存储库密码                                                   |
  | trans         | 转换的名称（在存储库中显示）设置为 发射                      |
  | dir           | 包含转换的存储库目录，包括 前导斜杠                          |
  | file          | 如果要调用本地 KTR 文件，则这是文件名，如果不在本地目录中时需要包括路径 |
  | level         | 日志记录级别（Basic、Detailed、Debug、Rowlevel、Error、Nothing） |
  | logfile       | 要将日志输出写入的本地文件名                                 |
  | maxloglines   | PDI 在内部保留的最大日志行数。设置为 0 保留所有行（默认）    |
  | maxlogtimeout | 保留日志行时的最长期限（以分钟为单位） 内部由 PDI 提供。设置为 0 可保留所有行 无限期（默认） |
  | listdir       | 列出指定存储库中的目录                                       |
  | listtrans     | 列出指定存储库目录中的转换                                   |
  | listrep       | 列出可用的存储库                                             |
  | exprep        | 将所有存储库对象导出到一个 XML 文件                          |
  | norep         | 阻止 Pan 登录到存储库。如果您已将 KETTLE_REPOSITORY、KETTLE_USER 和 KETTLE_PASSWORD 环境变量，然后这个 选项将使您能够阻止 Pan 登录到指定的存储库， 假设您想改为执行本地 KTR 文件。 |
  | safemode      | 在安全模式下运行，从而实现额外的检查                         |
  | version       | 显示版本、修订版本和构建日期                                 |
  | param         | 以某种格式设置命名参数。例如：name=value-param:FOO=bar       |
  | listparam     | 列出有关指定命名参数中定义的命名参数的信息 转型。            |

  示例: `sh pan.sh -rep=initech_pdi_repo -user=pgibbons -pass=lumburghsux -trans=TPS_reports_2011`

- Kitchen 是 PDI 用于 执行作业的命令行工具。

  用法: `kitchen.sh -option=value arg1 arg2`

  | Switch        | Purpose                                                      |
  | ------------- | ------------------------------------------------------------ |
  | rep           | 企业或数据库存储库名称（如果使用）                           |
  | user          | 存储库用户名                                                 |
  | pass          | 存储库密码                                                   |
  | **job**       | 要启动的作业的名称（在存储库中显示的名称）                   |
  | **dir**       | 包含作业的存储库目录，包括 前导斜杠                          |
  | file          | 如果要调用本地 KJB 文件，则这是文件名，如果不在本地目录中时需要包括路径 |
  | level         | 日志记录级别（Basic、Detailed、Debug、Rowlevel、Error、Nothing） |
  | logfile       | 要将日志输出写入的本地文件名                                 |
  | maxloglines   | PDI 在内部保留的最大日志行数。设置为 0 保留所有行（默认）    |
  | maxlogtimeout | 保留日志行时的最长期限（以分钟为单位） 内部由 PDI 提供。设置为 0 可保留所有行 无限期（默认） |
  | listdir       | 列出指定存储库中的目录                                       |
  | **listjob**   | 列出指定存储库目录中的作业                                   |
  | listrep       | 列出可用的存储库                                             |
  | **export**    | 导出指定作业的所有链接资源。参数是 一个 ZIP 文件。           |
  | norep         | 阻止 Pan 登录到存储库。如果您已将 KETTLE_REPOSITORY、KETTLE_USER 和 KETTLE_PASSWORD 环境变量，然后这个 选项将使您能够阻止 Pan 登录到指定的存储库， 假设您想改为执行本地 KTR 文件。 |
  | version       | 显示版本、修订版本和构建日期                                 |
  | param         | 以`name=value`格式设置命名参数。例如：`-param:FOO=bar`       |
  | listparam     | 列出有关指定命名参数中定义的命名参数的信息 转型。            |
  

示例: `sh kitchen.sh -rep=initech_pdi_repo -user=pgibbons -pass=lumburghsux -job=TPS_reports_2011`

#### 常用环境变量

| 名称                    | 默认值                |                                                              |
| ----------------------- | --------------------- | ------------------------------------------------------------ |
| KETTLE_HOME             | `~/`                  | 标识用户主目录的选项。该目录包含 配置文件，具体取决于登录的用户。<br />可以在`~/.kettle/`放置`kettle.properties`配置文件来设置变量（可以覆盖默认的变量或者设置自定义变量）。 |
| PENTAHO_DI_JAVA_OPTIONS | `-Xms1024m -Xmx2048m` | 在运行 Kettle 时传递其他 Java 参数的选项                     |

> 由于`pan.sh`和`kitchen.sh`这两个脚本也是直接使用调用`spoon.sh`，所以这些环境变量在这些命令中都是通用的。

### Pentaho Metadata Editor - 元数据编辑器

### Pentaho Report Designer - 报表设计器

### Pentaho Schema Workbench - 架构工作台

