
本篇又是一片笔记类文章，只记录尝试过程，不保证阅读通顺。

## 创建项目

在调查中发现，qiankun仓库就有一个最合适的[demo](https://github.com/umijs/qiankun/tree/master/examples)，然后计划在我的大而全的仓库[superapp](https://github.com/HelloWorld20/superapp)里新建一个package，来写qiankun demo，因为superapp里面已经有好几个前端服务，可以当做子应用。

仓库中新建：package/static-qiankun

把一些基础的代码搬到新package中：

```js
// packages/static-qiankun/index.js
import { start, registerMicroApps } from "qiankun";

import render from "./render/react-render";

render({ loading: true });

const loader = (loading) => render({ loading });

registerMicroApps([
  {
    name: "apart-radar",
    entry: "//localhost:3000",
    container: "#app",
    activeRule: "/apart-radar",
    loader, // loader貌似可以获取一些状态
  },
]);

start();
```

```js
// packages/static-qiankun/render/react-render.jsx
import React from "react";
import { createRoot } from "react-dom/client";

let root = null;

/**
 * 渲染子应用，qiankun需要一个入口。此处渲染一个<div id="subapp-viewport" />，子应用会挂载到这个节点上
 */
function Render(props) {
  const { loading } = props;
  return (
    <>
      {loading && <h4 className="subapp-loading">Loading...</h4>}
      <div id="app" />
    </>
  );
}

export default function render({ loading }) {
  // 有root则重复利用
  if (!root) {
    const container = document.getElementById("qiankun-app");
    root = createRoot(container);
  }
  root.render(<Render loading={loading} />);
}
```

结构几乎与qiankun官方demo一样。区别在于，superapp里用的react是18.2.0版本，createRoot方法不能重复调用。所以需要一个全局root对象来缓存root实例。重复使用createRoot方法返回的实例。（createRoot能否create不同的节点，实现不同根节点，如果不是，有什么意义，试试？）

### webpack

qiankun也是一个运行与浏览器的项目，正常运行也需要启动服务器来运行，也可以借助框架如React、Vue来渲染页面。

我们需要一个常规的webpack配置、配置dev-server、html-webpack-plugin等。来运行qinakun主应用，如果需要React，还需要加载babel-loader + @babel/plugin-transform-react-jsx等配置来支持。

详细配置就不贴了，不少代码。

因为是要测试模块联邦，所以用上了webpack5，此处就遇到不少坑。。这个坑官方啥时候填？？

首先是，无法读取子应用的生命周期。需要手动挂载

## 从官方demo改造

尝试过多次，未果，从另一个方向尝试。那就是用qiankun官方demo修改。

官方提供的demo用的是react16版本，webpack4。首先吧react16的代码复制一份，修改成react18，启动子应用，访问子应用。没问题。

但是，放到主应用，就开始有问题了。。具体表现如下：

1. 首次访问的是react18应用，会白屏。确定已经执行到mount方法，没有报错。
2. 首次访问的是react16应用，不会白屏，切换到react18应用，能正常渲染。
3. 在正常访问的情况下，来回切换react16与18应用。第二次切换到react18应用时，报错，并且显示loading，会执行18的unmount方法，报错：`Uncaught Error: Cannot update an unmount root.`

而且，不知道哪里来的一个z-index为2147483647的iframe遮挡住了整个页面。很奇怪。

再尝试回退一下，把createRoot方法改为ReactDOM.render。虽然有React18不建议使用render方法的warning，但是可以正常访问了。

所以结论是：qiankun与React18的createRoot方法水土不服。

## webpack5

既然顺利，那就继续在qiankun demo上修改。
```shell
cd examples/main

yarn add webpack@latest webpack-cli@latest webpack-dev-server@latest

yarn add html-webpack-plugin@latest
```

并且经过一点点小改动，可以正常跑起来，与webpack4没有任何区别。

### 模块联邦








