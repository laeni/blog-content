1. 在命令行启动后，按`R`可重新加载修改后的变更。
2. 按`Cmd/Ctrl + M`或摇动设备以打开 React Native 调试菜单。

# Api

## PropsWithChildren

声明 Props 中接收 `children`。

```typescript
type SectionProps = PropsWithChildren<{title: string}>;

function Section({children, title}: PropsWithChildren<{title: string}>): JSX.Element {
  return (
    <>
    </>
  );
}
```

# UI

## Flexbox

> Flexbox 在 React Native 中的工作方式与在 Web CSS 中的工作方式相同，但有一些例外。 默认值不同，有 `flexDirection`默认为 `column`代替 `row`,  `alignContent`默认为 `flex-start`代替 `stretch`,  `flexShrink`默认为 `0`代替 `1`， 这 `flex`参数只支持单个数字。 

### flex

[`flex`](https://reactnative.dev/docs/layout-props#flex)将定义项目如何沿主轴**“填充”**可用空间。空间将根据每个元素的 flex 属性进行划分。 

在下面的示例中，红色、橙色和绿色视图都是已设置`flex: 1`的容器视图中的子视图。红色视图使用`flex: 1`，橙色视图使用`flex: 2`，绿色视图使用`flex: 3`。而**1+2+3 = 6**，这意味着红色视图将获得`1/6`空间，橙色获得`2/6`的空间，绿色获得`3/6`的空间。

```typescript
import React from 'react';
import {StyleSheet, View} from 'react-native';

const Flex = () => {
  return (
    <View
      style={[
        {flex: 1, padding: 20},
        {
          // Try setting `flexDirection` to `"row"`.
          flexDirection: 'column',
        },
      ]}>
      <View style={{flex: 1, backgroundColor: 'red'}} />
      <View style={{flex: 2, backgroundColor: 'darkorange'}} />
      <View style={{flex: 3, backgroundColor: 'green'}} />
    </View>
  );
};

export default Flex;
```

### flexdirection

控制项目在坐标轴上的排列方向。可用值如下：

- `column` (**默认值**）从上到下对齐子项。如果启用了包装，则下一行将从容器顶部第一项的右侧开始。
- `row`从左到右对齐子项。如果启用了包装，则下一行将在容器左侧的第一项下开始。
- `column-reverse`从下到上对齐子项。如果启用了包装，则下一行将从容器底部第一项的右侧开始。
- `row-reverse`从右到左对齐子项。如果启用了包装，则下一行将从容器右侧的第一项开始。

### direction

指定层次结构中的子项和文本的布局方向。布局方向也会影响边缘`start`和`end`引用。默认情况下，React Native 使用 LTR 布局方向进行布局。在此模式下`start`指左，`end`指右。

- `LTR` (**默认值**）文本和子项从左到右排列。应用于元素开头的边距和填充将应用于左侧。
- `RTL`文本和子项从右到左排列。应用于元素开头的边距和填充将应用于右侧。