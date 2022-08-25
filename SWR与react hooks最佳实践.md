---
title: SWR与React Hooks最佳实践
date: 2022-07-26 11:49:00
tags: [总结,React]
---



在最近的一个需求中，第一次使用上[swr](https://swr.bootcss.com/)这个库。效果很棒，解决了不少痛点，并且也对React Hooks的使用有了更深一层的理解。

# 正文


## 概念

“SWR” 这个名字来自于 `stale-while-revalidate`：一种由 [HTTP RFC 5861](https://tools.ietf.org/html/rfc5861) 推广的 HTTP 缓存失效策略。这种策略首先从缓存中返回数据（过期的），同时发送 fetch 请求（重新验证），最后得到最新数据。

简单来说，就是，swr这个库会帮我们缓存get请求。并且封装成自定义hooks。看代码

```jsx
import useSWR from 'swr';

const fetcher = url => fetch(url).then(r => r.json())

function App () {
  const { data, error } = useSWR('/api/data', fetcher)
  // ...
}
```
这样，可以把远程的数据也可以以hooks的形式写在代码里。业务代码中只需关心data或者error更新时，做什么操作。

## 解决了什么问题

业务开发中，我们常常会把代码拆分成很多细小的模块。如，A组件下有两个子组件B、C。
B和C都需要请求一个接口。那么以前的做法是必须要把这个接口提升到B、C共同的父组件A上，在useEffect里发起请求。用useState存储接口数据，然后再用props传递给B、C组件。

如果需要调接口的子组件很多、且层级很深。那么往往需要跨很多层组件调接口、传props，props太多，代码耦合度就高了，也会产生很多无用的代码。代码会越来越难以维护

如果接口放到子组件则会产生额外的请求。

### swr如何解决的

swr可以把接口数据封装成hooks的形式，在需要的地方，插入一行代码即可注入。

还有一点，swr会缓存已经请求过的接口，所以该hooks被调用多少次、分散在多少个地方调用接口，都不会产生额外的请求。

## post更新数据

useSWR只适合get请求。更新数据则应该需要一个编程的方式更新

```jsx
import useSWR, { useSWRConfig } from 'swr';

const { mutate } = useSWRConfig();

mutate(updateFun(updateData));

```

## 更多高级用法

更多牛逼的用法，贴个链接

* [乐观更新](https://swr.bootcss.com/docs/mutation#%E4%B9%90%E8%A7%82%E6%9B%B4%E6%96%B0): 可以在发起更新请求时，乐观的直接更新本地数据。减少等待时间。
* [条件请求](https://swr.bootcss.com/docs/conditional-fetching): 可以很智能的链式依赖的条件请求数据，捕获promise错误
* [自动重新请求](https://swr.bootcss.com/docs/revalidation): 这个有点酷了，自动重新请求，窗口获得焦点、网络重新连接时重新请求，可以使得应用及时获得最新数据。及时更新状态。
* [自动错误重试](https://swr.bootcss.com/docs/error-handling#%E9%94%99%E8%AF%AF%E9%87%8D%E8%AF%95): 自动错误重试，不用处理错误

## 判断loading状态

useSWR不会直接返回loading状态。需要判断三种useSWR的三个返回值的状态来自行判断

三个返回值有这几种`搭配`(注意是搭配，不是有这几种值)

```jsx
function App () {
  const { data, error, isValidating } = useSWR('/api', fetcher)
  console.log(data, error, isValidating)
  return null
}
// console.log(data, error, isValidating)
undefined undefined true  // => 开始 fetching
undefined Error false     // => 结束 fetching，出现错误
undefined Error true      // => 开始重试
Data undefined false      // => 重试结束，得到数据
```

loading状态应该是`data !== undefined && error === undefined`

所以也有一个小小的限制。接口返回值不能是undefined


## React Hooks最佳实践

之前面试有被问过一个问题：Hooks与高阶组件有什么区别？我一脸懵逼，高阶组件不是一种组合方式吗、Hooks不是一套新的API，可以让函数组件存储数据吗？他们之间有毛关系？

现在我才彻底理解当时那个问题的含义。

他们本质上确实没有关系，但是两者都共同解决了一个问题。就是数据注入的问题。

关于React数据注入有props、context、高阶组件，现在多了一个Hooks。props与context都是最常规，最直接的数据注入，没啥好说。

### 高阶组件

高阶组件的数据注入最直观的是`react-redux`的connect方法。connect方法把store里的数据直接注入到某一个组件。这种方式的缺点是，我们总需要去考虑数据是来自父组件还是connect注入的。两者冲突了还得特别处理。

还有就是，我们往往需要写不少代码来正确处理数据、ref。

### hooks

hooks本质上也是一套新的数据注入的方式。最直观的也是`react-redux`的`useSelector`方法

我们要做的只是在需要数据的地方，插入一个自定义hooks，这个封装好的hooks会可靠的返回需要的数据，业务组件只需处理数据与视图的映射即可。

hooks的数据注入是`可插拔的热更新试`注入，对代码结构0影响。也无需额外代码。是一种更佳的数据注入方案。

转念一想想，这不就是hooks这个单词的本意？