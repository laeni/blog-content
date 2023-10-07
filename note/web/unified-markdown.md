---
title: 'unified生态下解析markdown'
author: 'Laeni'
tags: WEB, Markdown
date: '2022-02-13'
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

[**remark**](https://github.com/remarkjs/remark) （[*mdast*](https://github.com/syntax-tree/mdast)） — Markdown

[**retext**](https://github.com/retextjs/retext) （[*nlcst*](https://github.com/syntax-tree/nlcst)） — 自然语言

## 其他

[remark-rehype](https://unifiedjs.com/explore/package/remark-rehype/) - 将 Markdown 转换为 HTML

[remark-toc](https://unifiedjs.com/explore/package/remark-toc/) - 生成目录

[remark-gfm](https://github.com/remarkjs/remark-gfm) - [gfm（GitHub Flavored Markdown）](https://github.github.com/gfm/)是Markdown的方言，该插件将生成GitHub风格的HTML

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

## 示例

```tsx
import remarkGfm from "remark-gfm"
import remarkMath from 'remark-math'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import rehypeMathjax from 'rehype-mathjax'
import rehypeStringify from 'rehype-stringify'
import { unified } from 'unified'

const vfile = await unified()
	// 注意，这些插件顺序一般无关紧要，因为 unified 会自动根据插件的定义进行排序，但最终的效果大致类似于下面的顺序
    .use(remarkParse)     // 【解析】将 Markdown 解析为 mdast 语法树（markdown 语法树）
    .use(remarkMath)      // 【解析】对 remarkParse 未完全解析的数学公式（这里还是 Markdown 字符串）解析为 hast 语法树
    .use(remarkGfm)       // 【解析】GitHub 风格的 Markdown 支持（它能解析表格，原理同 remarkMath），如果不使用该插件，需要单独解析表格
    .use(remarkRehype)    // 【变换】将 mdast 语法树 转为 hast 语法树（HTML 语法树）
    .use(rehypeMathjax)   // 【变换】将 remarkMath 解析得到的部分特定 hast 语法树（数学公式部分）转换为 SVG
    .use(rehypeStringify) // 【编译】hast语法树 -> HTML 字符串
    .process('# Title\n\nMarkdown text.\n\nMath: $4=2^2$')
console.log('html: ', vfile.value)
```

> 注：React 中使用时一般使用[react-markdown](https://github.com/remarkjs/react-markdown)库，该库虽然是[unified](https://github.com/unifiedjs/unified)的包装，但是最终输出React组件，而不是普通HTML，这解锁了更多有用的东西，比如可以在Makdown中使用自定义的React组件等。

