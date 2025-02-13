# 概念

**conda:** [conda](https://docs.anaconda.com/working-with-conda/) 是 Anaconda 附带的命令行工具（CLI）。 

**Navigator:** [Navigator](https://docs.anaconda.com/navigator/) 是 Anaconda 附带的桌面图形用户界面 （GUI）。和 conda 一样可以启动应用程序和管理 conda 包、环境和通道，而无需使用命令行界面 （CLI） 命令。Navigator 可以在 Anaconda.org 上或本地 Anaconda 存储库中搜索包。

# Anaconda

Anaconda Distribution（Anaconda 发行版） 是一个免费的 Python/R 数据科学发行版，其中包含：

- [conda](https://conda.io/en/latest/)- 用于命令行界面的包和环境管理器
- [Navigator](https://docs.anaconda.com/navigator/)- Anaconda Navigator 是一个桌面应用程序，包含在每个 Anaconda Distribution 的安装包（不含 Minicinda）中。它建立在开源软件包和环境管理器 conda 之上，允许您从图形用户界面 （GUI） 管理软件包和环境。当您不熟悉命令行时，这尤其有用。
- 超过 250 种科学和机器学习[包](https://docs.anaconda.com/free/anaconda/pkg-docs/)

Anaconda Distribution 是免费的，易于安装，并提供[免费社区支持](https://community.anaconda.cloud/).要了解更多信息，请访问[开始使用 Anaconda Distribution](https://docs.anaconda.com/free/anaconda/getting-started/index.html).

## 安装与配置

安装 [Anaconda Distribution](https://docs.anaconda.com/anaconda/) 或 [Miniconda](https://docs.anaconda.com/miniconda/) 后，可以访问 conda、Python 和数以千计的其他流行软件包。Anaconda Distribution 会自动安装其中的 300 多个包，而 Miniconda 是一个更轻量级的发行版，仅包含 conda、python、它们的依赖项和少量其他包。参见[我应该使用 Anaconda Distribution 还是 Miniconda](https://docs.anaconda.com/distro-or-miniconda/)。

安装 [Anaconda Distribution](https://docs.anaconda.com/anaconda/install/) 版本 4.0.0 或更高版本时，会自动安装 Navigator。如果安装的是 [Miniconda](https://docs.anaconda.com/miniconda/) 或 Anaconda Distribution 版本 4.0.0 或更早版本，需要手动安装 Navigator：

```sh
$ conda install anaconda-navigator
```

### Docker

Docker 中也可以使用 [anaconda3](https://hub.docker.com/r/continuumio/anaconda3) 和 [miniconda](https://hub.docker.com/r/continuumio/miniconda3)。

```bash
$ docker run -i -t continuumio/miniconda3 /bin/bash
```

或

```bash
$ docker run -i -t -p 8081:8888 continuumio/miniconda3 /bin/bash -c "\
    conda install xxx
```

> 关于 Jupyter 的更多配置可参见 [Jupyter 专篇](./jupyter)。

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
  
- 安装中文语言包

  ```bash
  pip install jupyterlab-language-pack-zh-CN
  ```
  
  > 上述命令会将`jupyterlab-language-pack-zh-CN`包安装到当前激活的环境中，重启 jupyterlab 即可设置语言。

# Spyder

Spyder 是 Anaconda Distribution 附带的开源集成开发环境 （IDE），一般用于 Python 开发，提供高级编辑、交互式测试、调试和内省功能。

安装 Anaconda 后，可以从 Anaconda Navigator 可视化界面启动，也可以使用`spyder`命令启动。

> 如果是非`base`环境或者是 Mincinda，则一般需要新安装：
>
> ```sh
> $ conda install spyder
> ```
>
> 或者[独立安装](https://docs.spyder-ide.org/current/installation.html)（官方推荐）。

# Jupyter

## Jupyter Notebook

Jupyter Notebook 是一个基于网页的交互式计算应用程序，主要用于开发、文档编写、运行代码和展示结果。即它将代码、描述性文本、输出、图像和交互式界面合并到一个笔记本文件中，以便在 Web 浏览器中进行编辑、查看和使用

它支持多种编程语言，包括但不限于Python、R和Julia，使得用户可以在一个统一的环境中进行数据科学和科学计算工作‌。

Jupyter Notebook的主要功能包括：

- ‌**代码编辑和语法高亮**‌：在浏览器中进行代码编辑，自动进行语法突出显示，提供缩进和制表符自动完成/自检功能‌。
- ‌**直接运行代码**‌：从浏览器执行代码，并将计算结果直接附加到生成它们的代码上‌。
- ‌**富媒体结果展示**‌：使用 HTML、LaTeX、PNG、SVG 等格式展示计算结果，例如使用 matplotlib 库渲染的高质量图形‌。
- ‌**Markdown 和数学公式**‌：支持使用 Markdown 标记语言编辑富文本，并使用LaTeX轻松包含数学符号‌。
- ‌**文档存储和共享**‌：文档以.ipynb后缀的JSON格式存储，方便版本控制和共享，支持导出为HTML、LaTeX、PDF等格式‌。

Jupyter Notebook 的工作方式是通过 Jupyter Server 提供基于Web的界面和API服务，而代码片段的执行则由 Kernel 负责。浏览器通过 Http 和 Websockets 与 Jupyter Server 交互，Server 与 Kernel 通过 ZeroMQ 进行数据通信‌。

使用`jupyter-notebook`命令可以启动 Jupyter Notebook 并启动在浏览器打开，打开时默认会将运行命令的目录作为其『根目录』，可以在命令中带上路径修改根目录。

## JupyterLab

通过命令行启动**JupyterLab**：

```sh
$ jupyter-lab
```

# 环境

conda 支持多环境，环境是独立的隔离空间，可以在其中安装特定版本的软件包，包括依赖项、库和 Python 版本。

默认情况下，conda 默认启用的是 base（基本环境），而此环境是 conda 本身的安装位置，只能用于安装 anaconda、conda 和与 conda 相关的软件包，对于项目环境一般都需要创建新环境来使用，以免破坏 base 环境。

此处主要是 conda 命令的常见操作，可以查看[官方更完整列表和更全面的指南](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)。

## 列出已有的环境

方式一：

```sh
$ conda env list
```

方式二：

```sh
$ conda info --envs
```

## 创建环境

创建空环境：

```sh
$ conda create -n <ENV_NAME>
```

指定 Python 版本和包：

```sh
$ conda create -n <ENV_NAME> python=<VERSION> <PACKAGE>=<VERSION>
```

>如`conda create -n myenv python=3.11 beautifulsoup4 docutils jinja2=3.1.4 wheel`。

## 删除环境

```sh
$ conda env remove --name <ENV_NAME>
```

## 激活环境/切换环境

由于环境是隔离的，因此每次只能使用一个环境。当需要使用或切换另一个环境时只需要激活要是用的环境即可。

```sh
$ conda activate <ENV_NAME>
```

## 停用环境/取消激活

```sh
$ conda deactivate
```

> 最佳做法是在完成环境中的工作后停用环境。

## 共享环境

共享环境允许他人使用 conda 在其计算机上重新创建共享的环境。要共享环境及其软件包，必须将环境的配置导出到`.yml`环境配置文件中。

> 简单地将 Anaconda 或 Miniconda 文件复制到新目录或其他计算机不会重新创建环境。您必须将环境作为一个整体导出。

## 导出环境配置文件

```sh
$ conda env export > <ENV_NAME>.yml
```

> 将当前激活的环境的配置导出到`environment.yml`文件：
> 
> ```sh
> $ conda env export > environment.yml
> ```
>

## 从环境配置文件创建环境

```sh
$ conda env create -f <ENV_NAME>.yml
```

> 文件的第一行设置新环境的名称。有关更多详细信息，请参阅[手动创建环境文件](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually)。

# 包

## 共享包缓存

共享包缓存可以提高包的安装速度或新环境的创建速度，同时还能节省磁盘空间。默认情况下，包缓存目录在用户目录下，`conda info`输出中，`package cache`的值为包缓存的路径。

如需配置其他共享目录，可以在`~/.condarc`配置文件中添加以下内容:

```yaml
pkgs_dirs:
    - /path/to/shared_directory
```

## 安装包

可以使用`conda install`安装[Anaconda存储库](https://anaconda.org/)中的软件包，也可以使用`pip install`安装[Pip 包](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-pkgs.html#installing-non-conda-packages)。大部分情况下，它们可以协同工作，但如果 conda 软件包可用，则应该首选安装 conda 软件包。

**安装 conda 软件包**：

```sh
$ conda install package-name
```

**安装特定版本的 conda 软件包**：

```sh
$ conda install package-name=2.3.4
```

也可以**仅指定主要版本**：

```sh
$ conda install package-name=2
```

安装时，默认会将安装到当前活动的环境中，**如需安装到其他环境，可以指定环境名称**：

```sh
$ conda install package-name=2.3.4 -n some-environment
```

可以**从本地计算机直接安装 conda 包**（特别是离线环境可能很有用）：

```sh
$ conda install /package-path/package-filename.tar.bz2
```

如果需要同时安装多个离线包，可以将这些包归档成 .tar 文件（里面有一个或多个 .tar.bz2 包）再安装：

```sh
$ conda install /packages-path/packages-filename.tar
```

> **注意：**直接从文件安装包时不会解析依赖关系。如果已安装的软件包无法正常工作，则它可能缺少依赖项，需要手动解决。

如需将 R 语言与 Anaconda 一起使用，可参见[官方说明文档](https://docs.anaconda.com/working-with-conda/packages/using-r-language/)。

如需使用 GPU 包进行加速，可参见[官方说明文档](https://docs.anaconda.com/working-with-conda/packages/gpu-packages/)。
