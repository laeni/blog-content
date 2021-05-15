# 博客内容

该存储库专门用于存放博客 markdown 格式的内容，且默认情况下回忽略掉 README.md。

## 文章元数据格式

```typescript
export interface Matter {
    /**
     * 标题.
     */
    title: string;
    /**
     * 文章说明.
     */
    description?: string;
    /**
     * 作者.
     */
    author?: string;
    /**
     * 标签.
     * 字符串时以 ',' 分隔
     */
    tags?: string | string[];
    /**
     * 创建时间.
     */
    date?: string;
    /**
     * 更新时间.
     */
    updated?: string;

    /**
     * 开启文章的评论功能.
     * 默认: true
     */
    comments?: string;
}
```

## 说明

内容提交后，会自动触发流水线将内容构建为静态页面并部署。
