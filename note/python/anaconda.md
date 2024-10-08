# Anaconda


Anaconda Distribution（Anaconda 发行版） 是一个免费的 Python/R 数据科学发行版，其中包含：

- [conda](https://conda.io/en/latest/)- 用于命令行界面的包和环境管理器
- [Anaconda Navigator](https://docs.anaconda.com/navigator/)- 基于 conda 构建的桌面应用程序，具有从托管环境启动其他开发应用程序的选项
- 超过 250 种科学和机器学习[包](https://docs.anaconda.com/free/anaconda/pkg-docs/)

Anaconda Distribution 是免费的，易于安装，并提供[免费社区支持](https://community.anaconda.cloud/).要了解更多信息，请访问[开始使用 Anaconda Distribution](https://docs.anaconda.com/free/anaconda/getting-started/index.html).

## 安装与配置

安装时从[下载页面](https://www.anaconda.com/download)下载对应系统的安装包安装即可。

### 配置

- 一般情况下，每次通过 Shell 使用 Annaconda 前都需要进行初始化（例如将 Annaconda 路径添加到 $PATH），使用以下命令可以使 Annaconda 自动修改 Shell 配置文件，将初始化脚本添加到配置文件中。

  ```sh
  $ conda init --reverse $SHELL
  ```

  > 一般安装后会自动对当前使用的 Shell 进行设置，如需设置其他 Shell 可以手动初始化。

- 安装后 Annaconda 后，每次启动 Sheell 时都会自动激活 `base` 环境，激活环境后会改变命令行外观，如需禁用该行为可使用如下命令，同理，如果需要其自动设置时，可以将 `auto_activate_base` 设置为 `true`。

  ```sh
  $ conda config --set auto_activate_base false
  ```

## 概念

### 环境

列出已有的环境：

```sh
$ conda env list
```

创建环境：

```sh
$ conda create --name example
```

> 还可以指定 Python 版本：`conda create --name NAME [python=x.x]`。

删除环境:

```sh
$ conda env remove --name example
```

切换环境：

```sh
$ conda activate example
```

取消激活当前环境：

```sh
$ conda deactivate
```

### 安装包

conda install packge1 [packge2]

```sh
$ conda install pandas
```

### Jupyter

Jupyter Notebook 和 JupyterLab

通过命令行启动**JupyterLab**：

```sh
$ jupyter-lab
```

上述命令会启动一个 Jupyter 服务，端口默认为 8888，如果端口被占用则会依次递增寻找一个未占用的端口，并将当前目录作为工作目录。

# 足迹

1. [下载](https://www.anaconda.com/download)并安装。

2. 安装完成后进入[安装成功](https://www.anaconda.com/installation-success?source=installer)_（官方推荐将其存为书签）。

3. 从**安装成功**来到 [Anaconda Cloud](https://anaconda.cloud/)。

4. 从[Anaconda Cloud](https://anaconda.cloud/)来到课程[Anaconda 简介](https://freelearning.anaconda.cloud/get-started-with-anaconda)。

5. 

