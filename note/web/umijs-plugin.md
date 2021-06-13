---
title: '@umijs/plugin-dva 的相关概念和使用笔记'
author: 'Laeni'
tags: umijs, @umijs/plugin-dva, dva, react
date: '2021-06-13'
updated: '2021-06-14'
---
## @umijs/plugin-dva

dva 是一个基于 [redux](https://github.com/reduxjs/redux) 和 [redux-saga](https://github.com/redux-saga/redux-saga) 的数据流方案，为了简化开发体验，dva 额外内置了 [react-router](https://github.com/ReactTraining/react-router) 和 [fetch](https://github.com/github/fetch)。`@umijs/plugin-dva`目的是能在umi中快速集成dva。

[官方文档](https://umijs.org/zh-CN/plugins/plugin-dva) [DvaJS](https://dvajs.com/)

### 数据流向

数据的改变发生通常是通过用户交互行为或者浏览器行为（如路由跳转等）触发的，当此类行为会改变数据的时候可以通过 `dispatch` 发起一个 action，如果是同步行为会直接通过 `Reducers` 改变 `State` ，如果是异步行为（副作用）会先触发 `Effects` 然后流向 `Reducers` 最终改变 `State`，所以在 dva 中，数据流向非常清晰简明，并且思路基本跟开源社区保持一致（也是来自于开源社区）。

![img](https://zos.alipayobjects.com/rmsportal/PPrerEAKbIoDZYr.png)

### Model

namespace: model 的命名空间，同时也是他在全局 state 上的属性，只能用字符串，不支持多层命名空间

state: 存储数据，类似于React的State

reducers: 以 key/value 格式定义 reducer。用于处理同步操作，唯一可以修改（由 `action` 触发） `state` 的地方。

effects: 以 key/value 格式定义 effect。用于处理异步操作和业务逻辑，不直接修改 `state`。由 `action` 触发，可以触发 `reducer`，可以和服务器交互，可以获取全局 `state` 的数据等等

subscriptions: 以 key/value 格式定义 subscription。subscription 是订阅，用于订阅一个数据源，然后根据需要 dispatch 相应的 action。在 `app.start()` 时被执行，数据源可以是当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等

```typescript
// 示例 models/products.js
export default {
  // 命名空间
  namespace: 'index',
  // 保存数据
  state: { name: '' } as State,
  // 唯一修改 state 的地方
  reducers: {
    // 没启用 immer 之前
    // save(state, action) {
    //   return { ...state, ...action.payload };
    // },
    // 启用 immer 之后
    save(state: State, action: AsyncAction) {
      state.name = action.name;
    },
  } as { save: ImmerReducer<IndexModelState>; }, // 启用 immer 之前为 Reducer<IndexModelState>
  effects: {
    * query(action: AsyncAction, effectsCommandMap: EffectsCommandMap) { /* ... */ },
    * addRemote({ name }, { put, call }) {
      yield call(addTodo, name);
      yield put({ type: 'name', name });
    },
  } as { query: Effect; addRemote: Effect; },
  subscriptions: {
    setup({ dispatch, history }) {
      return history.listen(({ pathname }: any) => {
        if (pathname === '/') {
          // 在 model 内调用，不需要添加 namespace
          dispatch({ type: 'query', });
        }
      });
    },
  } as { setup: Subscription }
};
```

#### State

```typescript
type State = any
```

State 表示 Model 的状态数据，通常表现为一个 javascript 对象（当然它可以是任何值）；操作的时候每次都要当作[不可变数据（immutable data）](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/ch3.md#reasonable)来对待，保证每次都是全新对象，没有引用关系，这样才能保证 State 的独立性，便于测试和追踪变化。

#### Action

```typescript
type AnyAction = {
    // type 的值为 reducers 或者 effects 中的方法名,格式为: namespace/name
  	type: string;
    // 允许在 AnyAction 中定义任何额外的属性。
  	[extraProps: string]: any;
}
```

Action 是一个普通 javascript 对象，用于指明需要进行的操作(通过`type` 属性)以及放该操作中需要用到的数据(可以自定义其他字段)。无论是从 UI 事件、网络回调，还是 WebSocket 等数据源所获得的数据，最终都会通过`dispatch` 函数发起一个 action，从而改变对应的数据。需要注意的是 `dispatch` 是在组件 connect Models以后，通过 props 传入的。

```javascript
// 示例
dispatch({ type: 'index/save', name: 'Laeni' });
```

#### dispatch 函数

```typescript
type dispatch = (a: Action) => Action
```

dispatch 函数用于发射 action，action 是改变 State 的唯一途径，但是它只描述了一个行为，而 dipatch 可以看作是触发这个行为的方式，而 Reducer 则是描述如何改变数据的。

在 dva 中，connect Model 的组件通过 props 可以访问到 dispatch，可以调用 Model 中的 Reducer 或者 Effects，常见的形式如：

```javascript
dispatch({
  type: 'user/add', // 如果在 model 外调用，需要添加 namespace
  ...{ /* 需要传递的信息 */ }
});
```

#### Reducer

```typescript
type Reducer<S = any, A extends Action = AnyAction> =
        (state: S | undefined, action: A) => S;

// 这是采用了 immer 框架,即使在此处直接修改 state 也不会违反 Reducer 的规则
type ImmerReducer<S = any, A extends Action = AnyAction> =
        (state: S, action: A) => void;
```

Reducer（也称为 reducing function）函数接受两个参数：state 为该model改变前的数据，action 为前面所讲的一个行为描述，返回的是一个新的累积结果。即该函数把 state 和 action 合并成一个新 state。

需要注意的是 Reducer 必须是[纯函数](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/ch3.md)，所以同样的输入必然得到同样的输出，它们不应该产生任何副作用。并且，每一次的计算都应该使用[不可变数据（immutable data）](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/ch3.md#reasonable)，这种特性简单理解就是每次操作都是返回一个全新的数据（独立，纯净），所以热重载和时间旅行这些功能才能够使用。

#### Effect

```typescript
type Effect = (action: AnyAction, effects: EffectsCommandMap) => void;
```

Effect 被称为副作用，在我们的应用中，最常见的就是异步操作。它来自于函数编程的概念，之所以叫副作用是因为它使得我们的函数变得不纯，同样的输入不一定获得同样的输出。

dva 为了控制副作用的操作，底层引入了[redux-sagas](http://superraytin.github.io/redux-saga-in-chinese)做异步流程控制，由于采用了[generator的相关概念](http://www.ruanyifeng.com/blog/2015/04/generator.html)，所以将异步转成同步写法，从而将effects转为纯函数。至于为什么我们这么纠结于**纯函数**，如果你想了解更多可以阅读[Mostly adequate guide to FP](https://github.com/MostlyAdequate/mostly-adequate-guide)，或者它的中文译本[JS函数式编程指南](https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details)。

#### EffectsCommandMap

1. actionChannel: *ƒ actionChannel(pattern, buffer)*

2. all: *ƒ all(effects)*

3. apply: *ƒ apply(context, fn)*

4. call: *ƒ call(fn)* - 用于调用异步逻辑，支持Promise

   ```typescript
   const result = yield call(fetch, '/todos');
   ```

   这个call与JS的call用法大概一致，第一个参数是要调用的函数，第二个参数开始是要传递给被调函数的参数，可传递多个。

5. cancel: *ƒ cancel()*

6. cancelled: *ƒ cancelled()*

7. cps: *ƒ cps(fn)*

8. flush: *ƒ flush(channel)*

9. fork: *ƒ fork(fn)*

10. getContext: *ƒ getContext(prop)*

11. join: *ƒ join()*

12. put: *ƒ put(action)* - 用于触发action

    ```typescript
    yield put({ type: 'todos/add', payload: 'Learn Dva'});
    ```

13. race: *ƒ race(effects)*

14. select: *ƒ select(selector)* - 用于从state里获取数据

    ```typescript
    const todos = yield select(state => state.todos);
    ```

15. setContext: *ƒ setContext(props)*

16. spawn: *ƒ spawn(fn)*

17. take: *ƒ take(type)*

18. takeEvery: *ƒ takeEvery(patternOrChannel, worker)*

19. takeLatest: *ƒ takeLatest(patternOrChannel, worker)*

20. takem: *ƒ ()*

21. throttle: *ƒ throttle(ms, pattern, worker)*

#### Subscription

Subscriptions 是一种从 **源** 获取数据的方法，它来自于 elm。

Subscription 语义是订阅，用于订阅一个数据源，然后根据条件 dispatch 需要的 action。数据源可以是当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等。

```typescript
import key from 'keymaster';
...
app.model({
  namespace: 'count',
  subscriptions: {
    keyEvent({dispatch}) {
      key('⌘+up, ctrl+up', () => { dispatch({type:'add'}) });
    },
  }
});
```

### umi 接口

常用方法可从 umi 直接 import。

比如：`import { connect } from 'umi';`

接口包含:

#### connect

绑定数据到组件。

#### getDvaApp

获取 dva 实例，即之前的 `window.g_app`。

#### useDispatch

hooks 的方式获取 dispatch，dva 为 2.6.x 时有效。

#### useSelector

hooks 的方式获取部分数据，dva 为 2.6.x 时有效。

#### useStore

hooks 的方式获取 store，dva 为 2.6.x 时有效。

### 类型

通过 umi 导出类型：`ConnectRC`，`ConnectProps`，`Dispatch`，`Action`，`Reducer`，`ImmerReducer`，`Effect`，`Subscription`，和所有 `model` 文件中导出的类型。

### 其他

#### immer.js

[Immer](https://github.com/mweststrate/immer) 是 mobx 的作者写的一个 immutable 库，核心实现是利用 ES6 的 proxy，几乎以最小的成本实现了 js 的不可变数据结构，简单易用、体量小巧、设计巧妙，满足了我们对JS不可变数据结构的需求。

简单使用和介绍见:  <https://segmentfault.com/a/1190000017270785>

#### Router

这里的路由通常指的是前端路由，由于我们的应用现在通常是单页应用，所以需要前端代码来控制路由逻辑，通过浏览器提供的 [History API](http://mdn.beonex.com/en/DOM/window.history.html) 可以监听浏览器url的变化，从而控制路由相关操作。

dva 实例提供了 router 方法来控制路由，使用的是[react-router](https://github.com/reactjs/react-router)。

```jsx
import { Router, Route } from 'dva/router';
app.router(({history}) =>
  <Router history={history}>
    <Route path="/" component={HomePage} />
  </Router>
);
```

#### Route Components

在[组件设计方法](https://github.com/dvajs/dva-docs/blob/master/v1/zh-cn/tutorial/04-组件设计方法.md)中，我们提到过 Container Components，在 dva 中我们通常将其约束为 Route Components，因为在 dva 中我们通常以页面维度来设计 Container Components。

所以在 dva 中，通常需要 connect Model的组件都是 Route Components，组织在`/routes/`目录下，而`/components/`目录下则是纯组件（Presentational Components）。
