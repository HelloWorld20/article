
## 创建qiankun

首先qiankun主应用也是一个常规应用。也可能需要渲染UI，所以也可能需要用React、Vue等框架来渲染。

1. 所以我们需要一个常规的webpack配置、配置dev-server、html-webpack-plugin等。
2. 调用`registerMicroApps`和`start`方法，启动子应用。
3. 运行主应用，则qiankun会根据配置信息，把子应用加载到指定的容器中。

额外的，我们也可以在主应用入口处渲染主应用的UI。

接下来我们需要在主应用的入口加载子应用。加载主应用。

我们需要使用乾坤暴露出来的render方法

也就是调用`registerMicroApps`与`start`方法。

## 子应用的改造

需要在子应用的入口处导出三个生命周期函数，供qiankun调用

```jsx

export async function bootstrap() {
  console.log("bootstrap");
}

export async function mount(props: any) {
  console.log("mount");
  ReactDOM.createRoot(document.getElementById("app")!).render(<Route />);
}

export async function unmount() {
  console.log("unmount");
  unmountComponentAtNode(document.getElementById("app")!);
}

if (!(window as any).__POWERED_BY_QIANKUN__) {
  bootstrap().then(mount);
}

```

最后一个if是判断，如果不在qiankun环境中，则自己加载，渲染应用


其他

1.  子应用要转UMD（但是针对webpack5，此方法无用）
2. 针对webpack5，需要手动挂载生命周期，类似pureHTML挂载生命周期的方式（[webpack5挂载生命周期的问题](https://github.com/umijs/qiankun/issues/1092#issuecomment-1109673224)）
3. react18 ReactDOMClient.createRoot不能调用多次的问题


## webpack需要转出umd


## react-dom18不能重复调用createRoot问题

问题出在qiankun主应用。切换loading的过程，调用的render方法未经处理，则是重复调用createRoot。

解决办法也简单，新建一个全局root对象服用root对象即可

```jsx
let root = null;

function getRoot() {
  if (root) {
    return root;
  }
  const container = document.getElementById("qiankun-app");
  return (root = createRoot(container));
}

export default function render({ loading }) {
  const root = getRoot();

  root.render(<Render loading={loading} />);
}

```

## react18子应用不渲染问题

好家伙，完全不知道为什么。如果在

# 利用superapp框架重新创建

重新创建两个包，命名为static-qiankun-child和static-qiankun-main。并且新增必要文件，运行起来一个最精简的项目

## 安装依赖

	pnpm add qiankun -r --filter @ww/static-qiankun-main

	pnpm add webpack@latest webpack-cli@latest webpack-dev-server@latest html-webpack-plugin@latest -r --filter @ww/static-qiankun-*

	pnpm add webpack@latest webpack-cli@latest webpack-dev-server@latest html-webpack-plugin@latest -r --filter @ww/tools-build-react

写上最基础代码

```jsx
import { start, registerMicroApps } from "qiankun";

import render from "./render/react";

render({ loading: true });

const loader = (loading: boolean) => render({ loading });

registerMicroApps([
  {
    name: "child1",
    entry: "//localhost:3000",
    container: "#app",
    activeRule: "/child1",
    loader, // loader貌似可以获取一些状态
  },
]);

setDefaultMountApp("/child1")

start();
```

果不其然，报错：`[qiankun]: You need to export lifecycle functions in child1 entry`

手动挂载生命周期后，还是出现莫名其妙的空白！！！

区别在于当前demo用的npm上的qiankun，而之前的demo用的是qiankun源码。

尝试把源码搬过来。后面再debug怎么回事。

## 搬源码

新建一个package命名为：lib-qiankun-es

把qiankun demo里的es目录下代码复制过来，npm init -y，新增一个package.json。

运行后发现缺少不少依赖，其中还有@babel/runtime，是babel构建时插入的，没有写在package.json里。所以手动安装一波

single-spa、lodash、@babel/runtime、import-html-entry。之后服务可正常跑起来。

但是还是白屏！！！

难不成多包管理或者webpack不行？

继续找不同。

* 发现用qiankun官方demo启动主应用，superapp-qiankun启动子应用可以使用。
* 代码一模一样都没用，还是需要debug源码。

把引用指向源码，更方便debug。亲测不行，缺少loader。看了下输出可读性还算高。直接debug打包后代码。


## debug qiankun代码。

发现 start函数里的 startSingleSpa注释掉之后就能正常显示，不会覆盖子应用。

startSingleSpa来自single-spa的start方法。版本都是5.9.4，不是版本区别的问题。

还有，注释掉后，ReactDOMClient.createRoot的warning只调用一次，而未注释时，有三个warning。qiankun demo里没有warning？？？


不断debug，找到会清空子应用的具体代码

qiankun:start() => single-spa:start() => reroute() => performAppChanges() => tryToBootstrapAndMount() => toMountPromise() => reasonableTime() =>

	appOrParcel[lifecycle](getProps(appOrParcel)).then(val => {
	  finished = true;
	  resolve(val);  // 问题在这，这个resolve调用后则为空。
	}).catch(val => {
	  finished = true;
	  reject(val);
	});


再debug后，发现是resolve后会调用createRoot。再一次调用createRoot后会清空节点，导致空白。但是为什么qiankun-copy里没有问题呢？

因为没复制main/src/render/react.tsx里的代码。

复用createRoot或者ReactDOM.render都可以，但是不能重复createRoot，重复createRoot则会删除节点。

当然ReactDOM.render会报warning，已经不推荐。

## 总结

果然还是走了弯路，本质上只是重复调用ReactDOM.createRoot的问题。后面调用的createRoot会把原先的清空。




# 深究问题

* 为什么webpack5不能导出生命周期？
* 为什么npm版的qiankun导致子应用白屏？