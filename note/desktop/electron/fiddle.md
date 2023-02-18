electron **Fiddle** 可创建和运行小型 electron 示例。比如官方文档中的很多示例可以直接跳转到 Fiddle 中运行。一般将其安装为学习工具或在开发过程中试验Electron的API或特性功能。

# 使用

## 编辑器

Electron Fiddle 允许您使用 Electron 构建小实验和迷你应用程序。 每个 Fiddle 至少包含三个文件：一个主脚本、一个渲染器脚本、一个预加载脚本和一个 HTML 文件。

如果你`require()`一个模块，Fiddle 会自动安装它。 它还会自动为您提供电子模块的自动完成信息。

## 版本

Electron Fiddle 知道所有发布的 Electron 版本，在后台自动下载使用的版本。

打开首选项以查看所有可用版本并删除以前下载的版本。

## 分享Fiddle

可以将 Fiddle 保存为公共 GitHub Gist，允许其他用户通过将 URL 粘贴到地址栏来加载它。 如果他们没有 Electron Fiddle，他们可以直接从 GitHub 查看和下载您的代码。

您还可以将 Fiddle 打包为独立的二进制文件或“任务”菜单中的安装程序。

