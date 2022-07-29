---
title: 'Webpack5 Module Federation微前端初探'
author: 'Laeni'
tags: 'webpack5, Module Federation, 模块联邦'
date: '2021-06-20'
updated: '2021-06-20'
---

最近看到微前端相关的一些东西，就去对比了下很多微前端框架的实现思路和使用方式，虽然目前微前端技术还不是很成熟，但是也涌现出很多框架，比如蚂蚁的[qiankun](https://qiankun.umijs.org/)（基于[single-spa系](https://github.com/single-spa)）、[bit](https://harmony-docs.bit.dev/)和[webpack5 module-federation](https://webpack.js.org/concepts/module-federation/)等。

以上三个是我经过简单比较之后认为相对简单的微前端技术。其中`qiankun`解决的是应用级别的微前端框架，目的是帮助大家能更简单、无痛的构建一个生产可用微前端架构系统，但是实际体验下来也只能算是一般，毕竟考虑的多了之后需要解决的问题也就遇到，若是以后成熟后应该还是挺方便的。而`bit`和`webpack5 module-federation`都是组件级别的，且二者之间相似度很高，其中`bit`是将组件打包成普通 npm 包，通过分享包的方式实现组件的共享，使用项目可以直接像安装普通依赖一样使用发布的共享组件。`webpack5 module-federation`则是将组件打包为`bundle`后，通过加载不同的`bundle`使用不同的组件的方式实现共享。

由于`webpack5 module-federation`作为`webpack`的一项功能，所有它看起来更轻量级，且不限技术栈，只要能通过`webpack`打包就行。本笔记简单记录下`nextjs`下使用`webpack5 module-federation`的一些配置和相关代码，方便查阅。

## 相关的概念

**Remote**：被 Host 消费的 Webpack 构建。

**Host**：消费其他 Remote 的 Webpack 构建。

一个应用可以是 Host，也可以是 Remote，还可以同时是 Host 和 Remote。

## Remote 应用配置

nextjs 的Remote项目使用`@module-federation/nextjs-mf`插件可以方便配置，否则可能出现访问远程模块时出现 404，Host项目则可以不用。

```javascript
const { withModuleFederation } = require("@module-federation/nextjs-mf");

module.exports = {
    future: { webpack5: true },
    // https://www.nextjs.cn/docs/api-reference/next.config.js/custom-webpack-config
    webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack, ...options }) => {
        const mfConf = {
            mergeRuntime: true, //experimental
            name: "blog",
            library: {
                type: config.output.libraryTarget,
                name: "blog",
            },
            filename: "static/runtime/remoteEntry.js",
            exposes: {
                "./remoteName": "./components/RemoteName",
            },
            shared: ["lodash"],
            remotes: {
            },
        };
        config.cache = false;
        if (!isServer) {
            config.output.publicPath = "http://localhost:3000/_next/";
        }

        withModuleFederation(config, options, mfConf);
        return config;
    },
    webpackDevMiddleware: (config) => {
        // Perform customizations to webpack dev middleware config
        // Important: return the modified config
        return config;
    },
}
```

## Host 配置

```javascript
// const { withModuleFederation } = require("@module-federation/nextjs-mf");
const packageJsonDeps = require("./package.json").dependencies;

module.exports = {
  // 需要执行'next export'导出为静态页面时使用,目的是将图片的cdn地址设置为和本站一致（静态导出将失去图片优化）
  images: { loader: 'imgix', path: '/' },
  // 尾斜杠 - 将'/xxx.html'输出为'/xxx/index.html',以解决导出静态文件后直接访问子页面404的问题
  trailingSlash: true,
  future: { webpack5: true },
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack, ...options }) => {
    config.cache = false;
    // 该配置需要开启，否则动态加载时会有问题
    if (config.experiments) {
      config.experiments.topLevelAwait = true;
    } else {
      config.experiments = { topLevelAwait: true };
    }

    config.plugins.push(new webpack.container.ModuleFederationPlugin({
      remotes: {
        // 方式一: 如果打包时可以直接拿到该文件，则这里直接写死 Remote 应用导出的文件(!isServer 情况下,一般在全局html最前面引入该文件)
        // myremote: isServer ? '/home/laeni/Documents/xxx/.next/server/static/runtime/remoteEntry.js' : "myremote",
        // 方式二: 也可以直接通过http加载
        // myremote: "myremote@http://localhost:3000/_next/static/runtime/remoteEntry.js",
        // 方式三: 如果目前不能加载的话,可以先声明,否则打包时遇到引用该依赖时会报错
        // myremote: 'myremote',
        // 方式四: 如果可以通过动态配置的方式进行配置(包含该脚本文件地址)，则这里可以为空，这也是最大的好处，可以动态加载任意组件
      },
      shared: {
        ...packageJsonDeps,
        react: {
          eager: true, // we need this to fix "Uncaught Error: Shared module is not available for eager consumption"
          requiredVersion: packageJsonDeps.react,
          // singleton: true, we don't need singleton true because we are using React 17
        },
        "react-dom": {
          eager: true, // we need this to fix "Uncaught Error: Shared module is not available for eager consumption"
          requiredVersion: packageJsonDeps["react-dom"],
        },
      },
    }));

    return config;
  },

  webpackDevMiddleware: (config) => {
    // Perform customizations to webpack dev middleware config
    // Important: return the modified config
    return config;
  },
};
```

## Host & Remote 配置

略...

## 远程模块加载组件 - remoteNextMF.jsx

```jsx
import { useEffect, useState, lazy, memo, Suspense } from "react";
import Head from "next/head";

/**
 * 异步初始化并获取组件,由于是异步,所以需要hooks返回异步加载的组件.
 * 注意,由于将未渲染的组件放入hooks会导致取出来后无法使用,所有必须创建为react组件之后放入hooks.
 *
 * @param scope           区域
 * @param module          该区域下的某一个模块(组件)
 * @param props           该模块(组件)的props
 * @param setModuleFailed 设置模块的加载状态
 * @param ready           异步脚本的就绪状态(加载异步组件时需要保证对于的脚本已经正确加载)
 * @returns 加载后的组件(通过hooks异步返回)
 */
function loadComponent1(scope, module, props, setModuleFailed, ready) {
  const [component, setComponent] = useState(null);

  useEffect(async () => {
    if (!ready) {
      return
    }

    try {
      // 初始化
      await __webpack_init_sharing__("default");
      await global[scope].init(__webpack_share_scopes__.default);

      const Component = (await global[scope].get(module))().default;
      setComponent(<Component {...props}/>);
    } catch (e) {
      console.error(e);
      setModuleFailed(true);
    }
  }, [ready, module, scope]);

  return component;
}

/**
 * 通过异步组件简化组件获取(使用起来就像是同步一样).
 * TODO 注意: 多次使用会报错,该bug尚未解决.
 *
 * @param scope           区域
 * @param module          该区域下的某一个模块(组件)
 * @param props           该模块(组件)的props
 * @param setModuleFailed 设置模块的加载状态
 */
function loadComponent2(scope, module, props, setModuleFailed) {
  // 初始化
  try {
    global[scope].init(
      Object.assign(
        { react: { get: () => Promise.resolve(() => require("react")), loaded: true } },
        global.__webpack_require__ ? global.__webpack_require__.o : {}
      )
    );
  } catch (e) {
    console.error(e);
    setModuleFailed(true);
  }

  const Component = lazy(async () => (await global[scope].get(module))());

  return (
    <>
      {/*Component是通过懒加载加载进来的，所以渲染页面的时候可能会有延迟，但使用了Suspense之后，可优化交互*/}
      <Suspense fallback={<div>Loading...</div>}>
        <Component {...props}/>
      </Suspense>
    </>
  )
}

// const dynamicScripts = []; TODO 需要使用发布订阅来优化

function useDynamicScript(url) {
  // 标识是否准备就绪
  const [ready, setReady] = useState(false);
  // 表示是否失败
  const [failed, setFailed] = useState(false);

  useEffect(() => {
    // TODO 需要使用发布订阅来优化
    // if (dynamicScripts.includes(url)) { ... }
    // dynamicScripts.push(url);

    const element = document.createElement("script");
    element.src = url;
    element.type = "text/javascript";
    element.async = true;

    setReady(false);
    setFailed(false);

    element.onload = () => setReady(true);

    element.onerror = () => {
      setReady(false);
      setFailed(true);
    };

    document.head.appendChild(element);

    return () => {
      document.head.removeChild(element);
      // dynamicScripts.splice(dynamicScripts.indexOf(url), 1); TODO 需要使用发布订阅来优化
    };
  }, [url]);

  return { ready, failed };
}

/**
 * 动态加载 next 远程模块.
 */
export default memo(function RemoteNextMF({
  url,
  module,
  scope,
  mfProps = {}, // 需要传递给远程模块的参数
  errorComponent: ErrorComponent = () => "...共享模块加载出错...",
  loadingComponent: LoadingComponent = () => "...共享模块加载中...",
}) {
  if (!url || !scope || !module) {
    throw new Error("You must specify scope and module to import a Remote Component");
  }

  const { ready, failed } = useDynamicScript(url);
  const [moduleFailed, setModuleFailed] = useState(false);
  const component = loadComponent1(scope, module, mfProps, setModuleFailed, ready);
  // const component = ready && loadComponent2(scope, module, mfProps, setModuleFailed);

  return component ? component : failed || moduleFailed ? <ErrorComponent /> : <LoadingComponent />;
});
```

## 远程模块的使用

```jsx
import Head from "next/head";
import RemoteNextMF from '../components/remoteNextMF'


// 代码块使用动态加载,且服务端不能加载（Remote 模块导出时，如果使用方式四，即没有明确在构建是声明依赖的不能使用这种方式）
// const RemoteName = dynamic(() => import('blog/remoteName'), { ssr: false })

export default function Home({ loaded }) {
  return (
    <>
      <Head>
        <title>远程模块的使用</title>
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <RemoteNextMF
        url="http://localhost:3000/_next/static/runtime/remoteEntry.js"
        scope="blog"
        module="./remoteName"
        mfProps={{name: 'Laeni'}}
      />
    </>
  );
};
```



**P.S** 以上代码参考自[module-federation-examples](https://github.com/module-federation/module-federation-examples.git)，关于`module-federation`的更多用法可直接参考该项目。

## 其他

自动根据输出文件识别可用的组件正则： `("|')(\./[\w]+)("|'):[ ]?function\(\)`

## 参考资料

<https://survivejs.com/webpack/output/module-federation/>