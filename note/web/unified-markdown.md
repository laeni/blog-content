---
title: 'unified生态下解析markdown'
author: 'Laeni'
tags: 'unified,remark,markdown'
date: '2022-01-13'
updated: '2022-02-13'
---

[unified](https://github.com/unifiedjs/unified)是一个使用抽象语法树 （AST） 转换内容的项目。以解析Markdown为例认识和学习unified。

## 步骤

1. markdown ----([remark-parse](https://unifiedjs.com/explore/package/remark-parse/))----> [mdast](https://github.com/syntax-tree/mdast)语法树（markdown 专用语法树）
2. [mdast](https://github.com/syntax-tree/mdast)语法树 ----([remark-rehype](https://unifiedjs.com/explore/package/remark-rehype/))----> hast语法树（HTML 专用语法树）
3. [hast](https://github.com/syntax-tree/mdast)语法树 ----([rehype-stringify](https://unifiedjs.com/explore/package/rehype-stringify/))----> HTML字符串

> [remark](https://unifiedjs.com/explore/package/remark/) = [unified](https://unifiedjs.com/explore/package/unified/) + [remark-parse](https://unifiedjs.com/explore/package/remark-parse/) + [remark-stringify](https://unifiedjs.com/explore/package/remark-stringify/)
>
> [rehype](https://unifiedjs.com/explore/package/rehype/)  = [unified](https://unifiedjs.com/explore/package/unified/) + [rehype-parse](https://unifiedjs.com/explore/package/rehype-parse/) + [rehype-stringify](https://unifiedjs.com/explore/package/rehype-stringify/)

## 语法树

AST：抽象语法树，如 markdown 的抽象语法树是 mdast，html 的抽象语法树是 hast。

[**esast**](https://github.com/syntax-tree/esast) — JS

[**hast**](https://github.com/syntax-tree/hast) — HTML

[**mdast**](https://github.com/syntax-tree/mdast) — Markdown

[**nlcst**](https://github.com/syntax-tree/nlcst) — 自然语言

[**xast**](https://github.com/syntax-tree/xast) — XML

## 处理器列表

[**rehype**](https://github.com/rehypejs/rehype) （[*hast*](https://github.com/syntax-tree/hast)） — HTML

[**备注**](https://github.com/remarkjs/remark) （[*mdast*](https://github.com/syntax-tree/mdast)） — Markdown

[**retext**](https://github.com/retextjs/retext) （[*nlcst*](https://github.com/syntax-tree/nlcst)） — 自然语言

## 其他

[remark-rehype](https://unifiedjs.com/explore/package/remark-rehype/) - 将 Markdown 转换为 HTML

[remark-toc](https://unifiedjs.com/explore/package/remark-toc/) - 生成目录

[rehype-remark](https://unifiedjs.com/explore/package/rehype-remark/) - 将 HTML 转换为 Markdown

[rehype-slug](https://unifiedjs.com/explore/package/rehype-slug/) - 用于将id添加到标题中，添加后可以通过锚点导航到标题

[rehype-document](https://unifiedjs.com/explore/package/rehype-document/) - 将HTML片段（语法树）包装在完整的HTML文档中

[unist](https://github.com/syntax-tree/unist) - 语法树的规范

## Unist简介

Unist处理过程：

```
| .................................. process .................................. |
| .......... parse ...... | ........ run ........ | ....... stringify ..........|

          +-------------+                            +----------------+
Input ->- | Parser(解析) | ->- Syntax Tree(语法树) ->- | Compiler(编译) | ->- Output
          +-------------+             |              +----------------+
                                      X
                                      |
                             +-------------------+
                             | Transformers(变换) |
                             +-------------------+
```

如图所示，使用unist时必须有“解析”和“编译”这两个过程，否则会报错误。

