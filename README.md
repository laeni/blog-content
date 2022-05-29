# 博客内容

该存储库专门用于存放博客 markdown 格式的文章内容，且默认情况下会忽略掉 README.md。博客源码参见[bolog](https://github.com/laeni/blog)存储库.

master分支变动时会触发[阿里云-云效 Flow](https://flow.aliyun.com/)构建为静态页面并自动部署。

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
    comments?: boolean;

    /**
     * 是否隐藏.
     * 防止部分还为完成的文章不小心提交后对外展示.
     * 默认: false
     */
    hide?: boolean;
}
```
