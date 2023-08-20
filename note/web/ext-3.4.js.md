---
title: Ext.js 笔记
author: Laeni
tags: Ext, JavaScript, Jsp
deprecated: true
date: '2023-08-07'
updated: '2023-08-07'
---

当下，像**Ext.js**这类前端框架早已经过时，但是要维护老项目必须得懂一点，所以仅作为学习笔记记录。

# 项目搭建

```html
<!DOCTYPE html>
<html>

<head>
    <link href="CSS3.4.0/ext-all.css" type="text/css"/>
    <script src="ExtJs3.4.0/ext-base.js" type="text/javascript"></script>
    <script src="ExtJs3.4.0/ext-all.js" type="text/javascript"></script>
</head>

<body>
    <script>
        Ext.onReady(function () {
            console.log('Ext.onReady() 方法将在 Ext JS 准备好渲染 Ext JS 元素时调用')
        })
    </script>
</body>

</html>
```

# 常用函数

## Ext.onReady / Ext.EventManager.onDocumentReady

添加一个监听器，以便在文档准备好时（在加载之前和加载图像之前）收到通知。 Ext.EventManager.onDocumentReady 的简写。

## Ext.extend

类继承。

例如，继承`Ext.grid.GridPanel`类：

```javascript
MyGridPanel = Ext.extend(Ext.grid.GridPanel, {
    constructor: function(config) {
				// Create configuration for this Grid.
        var store = new Ext.data.Store({...});
        var colModel = new Ext.grid.ColumnModel({...});

        // Create a new config object containing our computed properties
        // *plus* whatever was in the config parameter.
        config = Ext.apply({
            store: store,
            colModel: colModel
        }, config);

        MyGridPanel.superclass.constructor.call(this, config);
				// Your postprocessing here
    },

    yourMethod: function() {
        // etc.
    }
});
```

### Parameters

- superclass : Function

  被继承的类。

- overrides : Object

  子类成员，如果和父类有冲突，则会覆盖父类的成员。

  如果存在“**constructor**”特殊函数时，父类的构造函数将会被它覆盖。但是**必须在提供的构造函数中调用超类构造函数（e.g. `Xxx.superclass.constructor.call(this[, ...])`）。**

  > 构造函数还可以通过第一个参数传递（将其作为三参数函数使用），但作为三参数函数使用时，`overrides`中的`constructor`属性将会被忽略，因为已经通过第一个参数指定了。
  >
  > ```javascript
  > MyGridPanel = Ext.extend(function() { /* TODO */ }, Ext.grid.GridPanel, {
  >     yourMethod: function() {
  >         // etc.
  >     }
  > });
  > ```

### Returns

- [Function](https://docs.sencha.com/extjs/3.4.0/#!/api/Function)

  来自`overrides`参数的子类构造函数，如果未提供，则为生成的构造函数。

## Ext.Panel

面板是一个具有特定功能和结构组件的容器，使其成为面向应用程序的用户界面的完美构建块。

面板由于继承自 Ext.Container，能够配置布局并包含子组件。

当指定面板的子项或动态地将组件添加到面板时，请记住考虑您希望面板如何排列这些子元素，以及是否需要使用 Ext 的内置布局方案之一来调整这些子元素的大小。 默认情况下，面板使用 ContainerLayout 方案。 这只是渲染子组件，将它们一个接一个地附加到容器内，并且根本不应用任何大小调整。

面板还可能包含底部和顶部工具栏，以及单独的页眉、页脚和正文部分（有关其他信息，请参阅框架）。

面板还提供内置的可扩展和可折叠行为，以及各种可以连接以提供其他自定义行为的预构建工具按钮。 面板可以轻松放入任何容器或布局中，并且布局和渲染管道完全由框架管理。

# 参考文件

- [官方API文档](https://docs.sencha.com/extjs/3.4.0/)
- [Ext.js_夜时的博客-CSDN博客](https://blog.csdn.net/qq_35007219/category_7394011.html)
