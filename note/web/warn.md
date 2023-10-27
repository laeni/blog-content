---
title: WEB 开发踩坑记
author: Laeni
tags: WEB, React
date: 2023-09-13
updated: 2023-09-13
---

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