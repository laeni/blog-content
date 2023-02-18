# 基础知识

## 主脚本

每个 Electron 应用程序都以一个主脚本开始，这与 Node.js 应用程序的启动方式非常相似。主脚本在“主进程”中运行。为了显示用户界面，主进程创建渲染进程（通常以窗口的形式，Electron 称之为 BrowserWindow）。

首先，假设主进程就像一个 Node.js 进程。 Electron 中的所有 API 和功能都可以通过 electron 模块访问，它可以像任何其他 Node.js 模块一样被导入。

## HTML

HTML 文件被加载到 BrowserWindow 中。 任何在浏览器中运行的 HTML、CSS 或 JavaScript 在这里也可以运行。 此外，Electron 允许您执行 Node.js 代码。 仔细查看`<script />`标签，注意我们如何像在 Node.js 中一样调用`require()`。

## 渲染脚本
这是前面 HTML 文件中需要的脚本。 在这里，您可以做任何在 Node.js 中工作的事情以及在浏览器中工作的任何事情。

顺便说一下：如果你想在这里使用一个 npm 模块，只需要`require`它。 Electron Fiddle 将自动检测您请求的模块并在您运行 fiddle 时立即安装它。

## Fiddle 示例

### main.js

```javascript
// Modules to control application life and create native browser window
const {app, BrowserWindow} = require('electron')
const path = require('path')

function createWindow () {
  // Create the browser window.
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // and load the index.html of the app.
  mainWindow.loadFile('index.html')

  // Open the DevTools.
  // mainWindow.webContents.openDevTools()
}

// This method will be called when Electron has finished initialization and is ready to create browser windows. Some APIs can only be used after this event occurs.
// 当 Electron 完成初始化并准备好创建浏览器窗口时，将调用此方法。 某些 API 只能在该事件发生后使用。
app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    // On macOS it's common to re-create a window in the app when the dock icon is clicked and there are no other windows open.
    // 在 macOS 上，当单击任务栏图标并且没有其他窗口打开时，通常会在应用程序中重新创建一个窗口。
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// Quit when all windows are closed, except on macOS. There, it's common for applications and their menu bar to stay active until the user quits explicitly with Cmd + Q.
// 当所有窗口关闭时退出，macOS 除外。 在那里，应用程序及其菜单栏通常会保持活动状态，直到用户使用 Cmd + Q 显式退出。
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})

// In this file you can include the rest of your app's specific main process code. You can also put them in separate files and require them here.
// 在此文件中，您可以包含应用程序的其余特定主进程代码。 您也可以将它们放在单独的文件中并在此处 require 它们。
```

### perload.js

```javascript
/**
 * The preload script runs before. It has access to web APIs
 * as well as Electron's renderer process modules and some
 * polyfilled Node.js functions.
 * 
 * https://www.electronjs.org/docs/latest/tutorial/sandbox
 */
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const type of ['chrome', 'node', 'electron']) {
    replaceText(`${type}-version`, process.versions[type])
  }
})
```

### index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'">
    <link href="./styles.css" rel="stylesheet">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.

    <!-- You can also require other files to run in this process -->
    <script src="./renderer.js"></script>
  </body>
</html>
```

### renderer.js

```javascript
/**
 * This file is loaded via the <script> tag in the index.html file and will
 * be executed in the renderer process for that window. No Node.js APIs are
 * available in this process because `nodeIntegration` is turned off and
 * `contextIsolation` is turned on. Use the contextBridge API in `preload.js`
 * to expose Node.js functionality from the main process.
 */
```

### styles.css

```css
/* styles.css */

/* Add styles here to customize the appearance of your app */
```

# 常见API

## electron.app

用于控制应用程序的事件生命周期。

# electron.app 常用事件

## ready - 就绪

一般就绪之后创建浏览器窗口，且它有一个便捷写法`app.whenReady().then(() => { ... })`。

```javascript
app.whenReady().then(() => {
  createWindow()
})
```

## window-all-closed - 窗口全部关闭

一般用于在非 Mac 下推出应用程序。

```javascript
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})
```

## activate -激活

由于 Mac 下全部关闭窗口不一定退出，所以当点击任务栏图标激活时需要创建窗口。

```javascript
app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) createWindow()
})
```

# 预加载脚本

Electron 的主进程是一个具有完全操作系统访问权限的Node.js环境。在[Electron](https://www.electronjs.org/docs/latest/api/app)模块之上，您还可以访问[Node.js内置模块](https://nodejs.org/dist/latest/docs/api/)， 以及通过 npm 安装的任何软件包。另一方面，渲染器进程运行 web 页面，并且出于安全原因默认不运行 Node.js。为了将 Electron 的不同类型的进程桥接在一起，我们需要使用被称为 **预加载** 的特殊脚本。

从 Electron 20 开始，预加载脚本默认**是[沙盒](https://www.electronjs.org/docs/latest/tutorial/sandbox)化的**，不再具有访问`require`权限和完整的 Node.js 环境。

| 可用的接口            | 详                                                           |
| --------------------- | ------------------------------------------------------------ |
| Electron 模块         | 渲染器进程模块                                               |
| Node.js 模块          | [`events`](https://nodejs.org/api/events.html), [`timers`](https://nodejs.org/api/timers.html), [`url`](https://nodejs.org/api/url.html) |
| Polyfilled 的全局模块 | [`Buffer`](https://nodejs.org/api/buffer.html), [`process`](https://www.electronjs.org/docs/latest/api/process), [`clearImmediate`](https://nodejs.org/api/timers.html#timers_clearimmediate_immediate), [`setImmediate`](https://nodejs.org/api/timers.html#timers_setimmediate_callback_args) |

预加载脚本像 Chrome 扩展的 [内容脚本](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)（Content Script）一样，会在渲染器的网页加载之前注入。 如果你想向渲染器加入需要特殊权限的功能，你可以通过 [contextBridge](https://www.electronjs.org/zh/docs/latest/api/context-bridge) 接口定义 [全局对象](https://developer.mozilla.org/en-US/docs/Glossary/Global_object)。

```javascript
/// preload.js
const { contextBridge } = require('electron');

contextBridge.exposeInMainWorld('versions', {
  node: () => process.versions.node,
  chrome: () => process.versions.chrome,
  electron: () => process.versions.electron,
});

/// renderer.js
console.log(`This app is using Chrome (v${versions.chrome()}), Node.js (v${versions.node()}), and Electron (v${versions.electron()})`)
```

#  [进程间通信](https://www.electronjs.org/zh/docs/latest/tutorial/ipc)

由于 Electron 的主进程和渲染进程有着清楚的分工并且不可互换。 这代表着无论是从渲染进程直接访问 Node.js 接口，亦或者是从主进程访问 HTML 文档对象模型 (DOM)，都是不可能的。

解决这一问题的方法是使用进程间通信 (IPC)。可以使用 Electron 的 `ipcMain` 模块和 `ipcRenderer` 模块来进行进程间通信。 为了从你的网页向主进程发送消息，你可以使用 `ipcMain.handle` 设置一个主进程处理程序（handler），然后在预处理脚本中暴露一个被称为 `ipcRenderer.invoke` 的函数来触发该处理程序（handler）。

我们将向渲染器添加一个叫做 `ping()` 的全局函数来演示这一点。这个函数将返回一个从主进程翻山越岭而来的字符串：

1. 首先，在预处理脚本（`preload.js`）中设置 `invoke` 调用：

   ```js
   const { contextBridge, ipcRenderer } = require('electron')
   
   contextBridge.exposeInMainWorld('versions', {
     ping: () => ipcRenderer.invoke('ping'),
   })
   ```
   
   > **IPC 安全：**
   >
   > 可以注意到我们使用了一个辅助函数来包裹 `ipcRenderer.invoke('ping')` 调用，而并非直接通过 context bridge 暴露 `ipcRenderer` 模块。 你**永远**都不会想要通过预加载直接暴露整个 `ipcRenderer` 模块，否则将使渲染器能够直接向主进程发送任意的 IPC 信息，会使其成为恶意代码最强有力的攻击媒介。
   
2. 然后，在主进程(`main.js`)中创建监听器。 我们在 HTML 文件加载*之前*完成了这些，所以才能保证在你从渲染器发送 `invoke` 调用之前处理程序能够准备就绪。

   ```js
   const { app, BrowserWindow, ipcMain } = require('electron')
   const path = require('path')
   
   const createWindow = () => {
     const win = new BrowserWindow({ width: 800, height: 600, webPreferences: { preload: path.join(__dirname, 'preload.js') } })
   
     win.loadFile('index.html')
   }
   
   // 监听 'ping' 事件
   ipcMain.handle('ping', () => 'pong')
   
   app.whenReady().then(createWindow)
   ```
   
3. 将发送器与接收器设置完成之后，现在可以在渲染进程(`renderer.js`)将信息通过刚刚定义的 `'ping'` 通道从渲染器发送至主进程当中。

   ```js
   const func = async () => {
     const response = await window.versions.ping()
     console.log(response) // 打印 'pong'
   }
   
   func()
   ```

# 常见问题

1. 如何理解上下文隔离以及默认情况下渲染进程无法使用原生 nodejs api？

   “渲染进程”顾名思义就是用于渲染视图，它就相当于一个控制台，所以大部分事情都不应该由它来完成，因为它的职责是显示“主进程”的状态，并且控制“主进程”，核心逻辑以及数据存储应该由主进程完成。

   > 注意这里打引号的主进程，即有些事情可能确实该由主进程来完成，但是考虑不能阻塞主进程等原因，实际任务主进程可能交给不用于渲染目的的渲染进程来完成。





















