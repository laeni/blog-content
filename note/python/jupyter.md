“Jupyter 项目”是一个大型伞形项目，涵盖许多不同的软件产品和工具。包括 [Jupyter Notebook](https://jupyter-notebook.readthedocs.io/en/latest/) 和 [JupyterLab](https://jupyterlab.readthedocs.io/en/latest/)，这两个程序都是流行的笔记本编辑器程序。**Jupyter 项目及其子项目都围绕着使用计算笔记本提供交互式计算的工具（和标准**）。JupyterLab 类似于一个 IDE 或 一个功能相对完善的平台，而 Jupyter Notebook 是 JupyterLab 的简化版本，其仅包含 Notebook 相关的核心功能。

# 概念

- Notebook

  笔记本是结合了计算机代码、简单语言的可共享文档 描述、数据、丰富的可视化效果，如 3D 模型、图表、图形和 图和交互式控件。一个笔记本和一个编辑器（如 JupyterLab） 为原型设计提供了一个快速的交互式环境，并且 解释代码、探索和可视化数据以及分享想法 别人。

# 安装/升级

[官方安装文档](https://docs.jupyter.org/en/latest/install.html)。

一般情况安装 Anacanda 后会同时安装好 Jupyter Notebook 和 JupyterLab，如果需要给多人使用可以通过 JupyterHub 安装。且一般情况安装 JupyterLab 即可，因为安装好 JupyterLab 后可以单独使用 Jupyter Notebook，但也可以仅安装 Jupyter Notebook（经典版）。

# jupyter-server

## 常用命名

### 输出 jupyter 相关的路径

```
jupyter --paths
```

### 生成默认配置

```sh
jupyter server --generate-config
```

### 生成/重置密码

```
jupyter server password
```

> 如果不设置密码则会自动生成一个 token，并在第一次登录时要求修改密码。

# Jupyverse

[Jupyverse](https://jupyter-server.github.io/jupyverse/) 是基于 [FastAPI](https://fastapi.tiangolo.com/) 的下一代 Jupyter 服务器。它可以代替 [jupyter-server](https://github.com/jupyter-server/jupyter_server/)，后者是 JupyterLab 默认安装的 Jupyter 服务器。

> 目前 Jupyverse 或社区还不成熟，所以如无特殊需求，使用默认的 jupyter-server 即可，因为目前 Jupyverse 无法直接使用密码（只能使用 token），如果需要使用密码则需要配合 [fps-auth-fief](https://jupyter-server.github.io/jupyverse/plugins/auth/#fps-auth-fief) 使用，而 fief 采用 python 开发，目前看起来相对较重，但是后续可能会有对应其他用户认证平台的插件实现 。不过可以[通过 Github 账号进行登录](https://jupyter-server.github.io/jupyverse/tutorials/jupyterhub_jupyverse_deployment/)。

## 安装

### conda

```sh
conda install -c conda-forge jupyverse fps-auth fps-jupyterlab fps-notebook
```

> `fps-lab`实现 JupyterLab 和 Jupyter Notebook 通用的所有内容，也可以明确同时安装两个`fps-jupyterlab`、`fps-notebook`。或仅安装其中一个。如果安装了 JupyterLab，则可以通过`/lab`访问；同理，如果安装了 Jupyter Notebook，则可以通过 `/tree`访问。

### pip

```sh
pip install "jupyverse[jupyterlab,notebook,auth]"
```

# Jupyter Notebook

[Jupyter Notebook](https://jupyter-notebook.readthedocs.io/en/latest/notebook.html) 是一个基于网页的交互式计算应用程序，主要用于开发、文档编写、运行代码和展示结果。即它将代码、描述性文本、输出、图像和交互式界面合并到一个笔记本文件中，以便在 Web 浏览器中进行编辑、查看和使用

它支持多种编程语言，包括但不限于Python、R和Julia，使得用户可以在一个统一的环境中进行数据科学和科学计算工作‌。

Jupyter Notebook的主要功能包括：

- ‌**代码编辑和语法高亮**‌：在浏览器中进行代码编辑，自动进行语法突出显示，提供缩进和制表符自动完成/自检功能‌。
- ‌**直接运行代码**‌：从浏览器执行代码，并将计算结果直接附加到生成它们的代码上‌。
- ‌**富媒体结果展示**‌：使用 HTML、LaTeX、PNG、SVG 等格式展示计算结果，例如使用 matplotlib 库渲染的高质量图形‌。
- ‌**Markdown 和数学公式**‌：支持使用 Markdown 标记语言编辑富文本，并使用LaTeX轻松包含数学符号‌。
- ‌**文档存储和共享**‌：文档以.ipynb后缀的JSON格式存储，方便版本控制和共享，支持导出为HTML、LaTeX、PDF等格式‌。

Jupyter Notebook 的工作方式是通过 Jupyter Server 提供基于Web的界面和API服务，而代码片段的执行则由 Kernel 负责。浏览器通过 Http 和 Websockets 与 Jupyter Server 交互，Server 与 Kernel 通过 ZeroMQ 进行数据通信‌。

使用`jupyter-notebook`命令可以启动 Jupyter Notebook 并启动在浏览器打开，打开时默认会将运行命令的目录作为其『根目录』，可以在命令中带上路径修改根目录。

> 安装好 Jupyter 后，虽然可以使用`jupyter notebook --notebook-dir=/data`方式启动，但是由于 JupyterLab 包含了 Jupyter Notebook，所以可以直接启动 JupyterLab 即可。

## Notebook 结构

Notebook 由一系列单元格（Cell）组成，单元格是多行文本输入字段。单元格可以执行，其执行行为由单元格的类型决定。有三个 单元格类型：**Code Cells（代码单元格）**、**Markdown Cells（Markdown 单元格）** 和 **Raw Cells（原始单元格）**。每单元格默认是一个**Code Cell**，但可以更改为其他类型。

可查看[示例集合](https://nbviewer.jupyter.org/github/jupyter/notebook/tree/main/docs/source/examples/Notebook/)获取更多信息。

### 代码单元格

*Code Cells*允许使用完整语法编写代码。使用的编程语言取决于内核，默认使用 IPython 内核运行 Python 代码。

执行代码单元时，它包含的代码将发送到与笔记本关联内核，并返回执行结果。然后执行结果将作为单元格*输出*显示在 Notebook 中。输出不限于文本，还有许多其他可能的输出形式，包括数字和 HTML 表格，这就是 IPython *丰富的显示*功能。

### Markdown 单元格

可以用一种文学的方式记录计算过程，将描述性文本与代码交替使用，并采用富文本格式。在 IPython 中，这是通过使用 Markdown 语言来标记文本实现的。相应的单元格被称为 Markdown 单元格。

如果希望为文档提供结构，可以使用 Markdown 标题。Markdown 标题将被转换为笔记本中该节的可点击链接。当导出到其他文档格式（如PDF）时，它也被用作提示。

### 原始单元格

原始单元格提供了一个可以直接书写输出的地方。原始单元格不会被 Notebook 评估。当通过[nbconvert]转换时，原始单元格将以未修改的形式到达目标格式。例如，你可以在原始单元格中键入完整的LaTeX代码，这些代码只有在经过nbconvert转换后才会被LaTeX渲染。

### 键盘快捷键

笔记本中的所有操作都可以使用鼠标执行，但键盘快捷方式也可用于最常见的快捷方式。基本快捷键为：

- Shift-Enter：运行单元 格：执行当前单元格，显示任何输出，然后跳转到下面的下一个单元格。 如果在最后一个单元格上调用，则会在下面创建一个新单元格。
- Esc：命令模式 ：在命令模式下，您可以使用键盘快捷键在笔记本中导航。
- Enter：编辑模式 ：在编辑模式下，您可以编辑单元格中的文本。

有关可用快捷方式的完整列表，请在 notebook 菜单中单击**帮助**,**键盘快捷键**。



# JupyterLab

通过命令行启动**JupyterLab**：

```sh
$ jupyter-lab
```

或

```sh
$ jupyter lab
```

> 有关启动参数参见[官方文档](https://jupyterlab.readthedocs.io/en/stable/getting_started/starting.html#starting-jupyterlab)。


- 安装中文语言包

  ```bash
  pip install jupyterlab-language-pack-zh-CN
  ```

  > 上述命令会将`jupyterlab-language-pack-zh-CN`包安装到当前激活的环境中，重启 jupyterlab 即可设置语言。

# JupyterHub

[JupyterHub](https://jupyterhub.readthedocs.io/) 是为多个用户提供 [Jupyter 笔记本](https://jupyter-notebook.readthedocs.io/en/latest/)的最佳方式。 由于 JupyterHub 为每个用户管理单独的 Jupyter 环境，因此 它可以用于学生班级、企业数据科学小组或科学 研究小组。它是一个多用户 **Hub**，可生成、管理和代理多个 单用户 [Jupyter 笔记本](https://jupyter-notebook.readthedocs.io/en/latest/)服务器的实例。

# 文档

## Docker Stacks

[Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/)是一组随时可运行的 [Docker 映像](https://quay.io/organization/jupyter)，其中包含 Jupyter 应用程序和交互式计算工具。

![Image inheritancediagram](jupyter.assets/inherit.svg+xml)

- [JupyterHub](https://jupyter.org/hub)
- [JupyterLab](https://jupyterlab.readthedocs.io/)
- [Jupyter Notbook](https://jupyter-notebook.readthedocs.io/)

