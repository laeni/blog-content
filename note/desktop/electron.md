## 工具

- [Electron Forge](https://electronforge.io/)：用于打包和分发Electron应用程序的多合一工具。
- [Electron Fiddle](https://www.electronjs.org/zh/fiddle)：用于快速进行实验的工具，可以理解为用于写Demo的。



Electron Fiddle 允许您使用 Electron 构建小示例和迷你应用程序。 每个 Fiddle 至少包含三个文件：一个主脚本(`main.js`)、一个渲染器脚本(`renderer.js`)、一个预加载脚本(`preload.js`)和一个 HTML 文件(`index.html`)。

如果你`require()`一个模块，Fiddle 会自动安装它。 它还会自动为您提供电子模块的自动完成信息。

---

每个 Electron 应用程序都以一个主脚本开始，这与 Node.js 应用程序的启动方式非常相似。 主脚本在“主进程”中运行。 为了显示用户界面，主进程创建渲染进程——通常以窗口的形式，Electron 称之为 BrowserWindow。

首先，假设主进程就像一个 Node.js 进程。 Electron 中的所有 API 和功能都可以通过 electron 模块访问，它可以像任何其他 Node.js 模块一样被要求。

默认的 fiddle 会创建一个新的 BrowserWindow 并加载一个 HTML 文件。

---

您可以将您的 Fiddle 保存为公共 GitHub Gist，允许其他用户通过将 URL 粘贴到地址栏来加载它。 如果他们没有 Electron Fiddle，他们可以直接从 GitHub 查看和下载您的代码。 您还可以将 Fiddle 打包为独立的二进制文件或“任务”菜单中的安装程序。

---

**HTML**: 在默认的 fiddle 中，这个 HTML 文件被加载到 BrowserWindow 中。 任何在浏览器中运行的 HTML、CSS 或 JavaScript 在这里也可以运行。 此外，Electron 允许您执行 Node.js 代码。 仔细查看` <script /> `标签，注意我们如何像在 Node.js 中一样调用`require()`。

**Renderer Script**: 这是我们刚刚从 HTML 文件中需要的脚本。 在这里，您可以做任何在 Node.js 中工作的事情以及在浏览器中工作的任何事情。顺便说一下：如果你想在这里使用一个 npm 模块，只需要`require()`它。 Electron Fiddle 将自动检测您请求的模块并在您运行 fiddle 时立即安装它。