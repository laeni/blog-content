---
title: Sublime Text 常用操作（主要是快捷键）
author: Laeni
tags: IDE
date: 2023-05-03
updated: 2023-05-04
---

# 快捷键

## 通用

- `↑↓←→` - 上下左右移动光标，注意不是不是 KJHL ！
- `Ctrl + Shift + P` - 调出命令板（Command Palette）
- `Ctrl + ~` - 打开/关闭控制台。
- `Ctrl + Z` - 撤销。
- `Ctrl + Shift + Z`/`Ctrl + Y` - 反撤销。
- `Ctrl + O` - 打开文件/目录。
- `Ctrl + Q` - 退出编辑器。

## 编辑

- `Ctrl + Enter` - 在当前行下面新增一行然后跳至该行

- `Ctrl + Shift + Enter` - 在当前行上面增加一行并跳至该行

- `Ctrl + ←/→` - 进行逐词移动

- `Ctrl + Shift + ←/→` - 进行逐词选择

- `Ctrl + ↑/↓` - 移动当前显示区域

- `Ctrl + Shift + ↑/↓` - 移动当前行

- `Ctrl + Shift + J` - 将选中的多行合并为一行。

  ![图片](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/ide/sublimetext/1.gif)

## 选择

- `Ctrl + D` - 选择当前光标所在的词并高亮该词所有出现的位置，再次`Ctrl + D`选择该词出现的下一个位置，在多重选词的过程中，使用`Ctrl + K`进行跳过，使用`Ctrl + U`进行回退，使用`Esc`退出多重编辑。
- `Alt + F3` - `Ctrl + D`的连续操作，该快捷键将全选鼠标所在词的所有位置。
- `Ctrl + L` - 每按一次就从鼠标所在行开始往下选择一行。
- `Ctrl + Shift + L` - 对选择的多行进行打散（在选中的每行中都插入光标），其效果类似于`Ctrl + Shift + Alt + 鼠标点选`。
- `Ctrl + M` - 在起始括号和结尾括号间切换。
- `Ctrl + Shift + M` - 可以快速选择括号间的内容。
- `Ctrl + Shift + Space` - 快速选择当前作用域（Scope）的内容

## 查找&替换

- `Ctrl + F` - 标准查找（将会选择查找到的内容）。
  - `Enter`向下查找。
  - `Shift + Enter`向上查找。
  - `Alt + Enter`查找全部
  - `Alt + C`切换大小写敏感（Case-sensitive）模式。
  - `Alt + W`切换整字匹配（Whole matching）模式。
  - `Alt + R`切换正则匹配模式。

- `Ctrl + H` - 标准替换（标准查找的升级版本，所有标准查找功能都适用）。
  - `Ctrl + Shift + H`替换当前关键字。
  - `Ctrl + Alt + Enter`替换所有匹配关键字。

- `Ctrl + Shift + F` - 开启多文件搜索&替换。默认在当前打开的文件和文件夹进行搜索/替换，我们也可以指定文件/文件夹进行搜索/替换。
- `F3` - 跳至当前关键字下一个位置
- `Shift + F3` - 跳到当前关键字上一个位置

## 跳转

- `Ctrl + P` - 跳转（Go To Anything）。会列出当前打开的文件（或者是当前文件夹的文件），输入文件名然后`Enter`跳转至该文件。Sublime Text使用模糊字符串匹配（Fuzzy String Matching），这也就意味着你可以通过文件名的前缀、首字母或是某部分进行匹配：例如， EIS 、 Eclip 和 Stupid 都可以匹配 EclipseIsStupid.java 。

  Sublime Text 还能进行**组合跳转**，即在`Ctrl + P`匹配到文件后，可以继续输入`@`（符号）、`#`（关键字）和`:`（行）以跳转到更精确的位置。

  ![组合跳转](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/ide/sublimetext/2.gif)

- `Ctrl + R` - 跳转到符号（类、接口、方法等）。等效于`Ctrl + P + @`。对于 Markdown，`Ctrl + R`会列出其大纲。

- `Ctrl + G` - 跳转到指定行。等效于`Ctrl + P + :`。

- `F12` - 快速跳转到当前光标所在符号的定义处（Jump to Definition）。

## 窗口

- `Ctrl + N` - 在当前窗口创建一个新标签。
- `Ctrl + Shift + N` - 创建一个新窗口。
- `Ctrl + W` - 关闭当前标签页。当窗口内没有标签时会关闭窗口。
- ` Ctrl + Shift + T` - 恢复刚刚关闭的标签。
- `Ctrl + K` - 组合快捷键，使用时先按`Ctrl + K`，然后再按子快捷键（有先后顺序）来触发功能。
  - `Ctrl + B` - 打开/隐藏**文件夹（Folders）**。可通过点击左下角图标实现。

## 屏幕

- `F11` - 切换普通全屏。建议开启全屏前关闭菜单栏（Toggle Menu），否则全屏效果会大打折扣。
- `Shift + F11` - 切换无干扰全屏。建议关闭自动换行。
- `Alt + Shift + <数字>` - 分屏。`1` - 取消分屏；`2` - 左右分 2 屏；`3` - 左右分 3 屏；`4` - 左右分 4 屏；`5` - 上下左右分屏（即分为 4 屏）；`8` - 上下分 2 屏；`9` - 上下分 3 屏。
  - `Ctrl + <数字>` - 跳转到数字指定的屏（仅在分屏模式下有效）。如`Ctrl + 2`跳转到第 2 屏。
  - `Ctrl + Shift + <数字>` - 将当前屏的内容移动到指定屏（仅在分屏模式下有效）。

# 可能用到的插件

- ChineseLocalizations - 简体中文语言。
- Terminus - 终端
- HTMLBeautify - 格式化HTML。
- AutoPEP8 - 格式化Python代码。
- Alignment - 进行智能对齐。
- BracketHighlighter - 插件以高亮显示配对括号以及当前光标所在区域

# 其他

Sublime Text [第三方主题](https://sublime.wbond.net/browse/labels/theme)。

# 参考

- [公众号文章](https://mp.weixin.qq.com/s/Y1N1oepRaQMHQeU_VgwjSQ)