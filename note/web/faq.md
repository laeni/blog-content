---
title: WEB开发常见问题及解决方案
description:
author: Laeni
tags: WEB, React, Vue, Angular
date: '2021-08-07'
updated: '2021-08-07'
---

# NextJs

## 不要重复安装`@ant-design/cssinjs`

`ant`已经包含了该已经，重复安装需要自己处理版本问题，如果`@ant-design/cssinjs`版本和`ant`版本不匹配会导致样式抽离失败。可使用`npm ls @ant-design/cssinjs`查看其版本关系。

# Ant Design

## 在 Dropdown（下拉）中使用 Popconfirm（气泡）时注意

由于点击 Dropdown 的选项时会导致 Dropdown 自动收起，Dropdown 的收起可能会销毁组件，进而导致其子组件也一起销毁，从而导致点击 Dropdown 选项中的 Popconfirm 组件时可能没反应（出现频率很高，但有时候也有效果）。为了解决该问题，需阻止 Popconfirm 组件的点击时间冒泡到上层组件即可。

```tsx
<Dropdown trigger={['click']} menu={{
  items: [
    { key: 'edit', label: '编辑', onClick: () => {} },
    {
      key: 'delete',
      danger: true,
      label: (
        <div onClick={e => e.stopPropagation()}>
          <Popconfirm okType='danger' title='确定要删除?' onConfirm={() => {}}>
            <a>删除</a>
          </Popconfirm>
        </div>
      )
    }
  ]
}}>
  菜单
</Dropdown>
```

# 其他

## 访问子页面且不带斜杠时Nginx会自动重定向到斜杠结尾的页面（常见于单页应用+预渲染）

