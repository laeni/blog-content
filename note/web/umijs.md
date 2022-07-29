Umi[官方文档](https://umijs.org/zh-CN/docs).

## 注意事项

### [HTML模板](https://umijs.org/zh-CN/docs/html-template)的存在会导致服务端渲染和预渲染失效

**问题**

HTML模板，即`src/pages/document.ejs`的存在会导致预渲染失效，因此预渲染也会随之失效。

当前已知问题版本为`3.4.25`。

**解决方案**

首先，使用HTML模板的主要目的有两个，一是设置HTML头标签，二是在js尚未加载完成时呈现等待提示（如动画）。

但在预渲染中，以上第二个目的已经通过服务端渲染或者预渲染达到，而第一个目的则可以通过配置，如[metas](https://umijs.org/zh-CN/config#metas)进行设置。还可通过`react-helmet`动态设置。