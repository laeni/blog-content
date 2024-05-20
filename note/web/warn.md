---
title: WEB 开发踩坑记，注意事项
author: Laeni
tags: WEB, React, Next.js
date: 2023-09-13
updated: 2024-03-16
---

# Next.js 刷新时路由查询参数为空

如[#10521](https://github.com/vercel/next.js/issues/10521)所述，这是刻意为之。官方这样做的理由大致是使用查询参数一般会请求后端等得到参数相关的数据，这会导致服务端渲染或静态生成时产生问题（生成产物与客户端产物不一致导致“水合”错误）。但我不这么认为，因为服务端渲染或静态生成时本来就不应该产生和客户端不一致的数据，即这个阶段不应该使用这些参数。事实上，我们通常也是这样做的，因为我们一般会在`useEffect` Hook 中使用参数获取数据。

解决方案：

- 自行解析参数

- 通过`router.isReady`状态判断是否可以使用查询参数。

  ```tsx
  export default function DemoPage() {
    const router = useRouter();
  
    useEffect(() => {
      // 查询参数为空时可能属于尚未准备好的情况，这时候什么也不做
      if (!router.isReady) {
        return;
      }
  
      TODO
    }, [router.isReady]);
  
    return ...
  }
  ```

# Next.js 中使用 @antv/g6 报错

由于不支持服务端渲染，所以报错，根据[#156](https://github.com/antvis/G6/issues/156)解决方案，如下示例：

```tsx
// 需要使用类型时仅仅导入类型（包括 Graph）
import type { EdgeConfig, Graph, INode, NodeConfig } from '@antv/g6';

export default function DemoPage() {
  useEffect(() => {
    (async () => {
      const { Graph } = await import('@antv/g6');

      const container = g6Container.current.current as HTMLDivElement;
      // 创建实例时使用的 Graph 实例必须使用动态导入的
      const graph = new Graph({
		TODO
      });

      // 设置画布的宽度
      canvas.width = graph.getWidth();
      canvas.height = graph.getHeight();
      // 添加节点事件
      addEvent(graph);
      updateGraphData(graph);
    })();
  }, []);

  return ...
}
```

# antd 表单数据回写没清空可选表单项

```tsx
const [editForm] = Form.useForm();

return (
    <>
        <div>
            <Button onClick={() => {
                editForm.setFieldsValue({ id: 1, name: '张三', age: 18 });
            }}>张三</Button>
            <Button onClick={() => {
                editForm.setFieldsValue({ id: 2, name: '李四' });
            }}>李四</Button>
            <Button onClick={() => {
                editForm.resetFields();
                editForm.setFieldsValue({ id: 3, name: '王五', age: 20 });
            }}>王五</Button>
        </div>

        <Form form={editForm}>
            <Form.Item label='id' name='id' hidden>
                <Input />
            </Form.Item>
            <Form.Item label='姓名' name='name'>
                <Input />
            </Form.Item>
            <Form.Item label='年龄' name='age'>
                <Input />
            </Form.Item>

            <Form.Item>
                <Button type='primary' htmlType='submit'>提交</Button>
            </Form.Item>
        </Form>
    </>
);
```

考虑以上代码，我们时常会使用一个表单回显多条逻辑独立的数据，在这个示例中，当先点击“**张三**”再点击“**李四**”时由于`age`字段为空导致“**李四**”的编辑表单中依然显示`18`，而不是我们期望的空。这是因为`setFieldsValue`方法的功能仅仅是**批量设置**表单字段，所以没有的字段就认为**不设置**它，这就导致这个问题。

修复它的方法有两种，一种是像例子中的“**李四**”那样，设置值前先重置一下；还有一种就是为每个用户提供一个独享的表单实例（常用方法是将`From`表单选择提出为一个单独的组件，每个组件实例对应一个用户数据）。



