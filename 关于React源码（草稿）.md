---
title: 关于React源码学习（草稿）
date: 2020-11-21 17:05:24
categories: 
    - 总结,React
tags: 
    - 总结,React
---

最近尝试学习React的源码，学习阶段，先建个草稿，记录片段。

# 重要学习资料

[React技术揭秘-卡颂](https://react.iamkasong.com/)


# 正文

个人觉得，React很多处代码都是异步执行的，不能像其他仓库一样从入口文件开始调试。学习React源码则得从好几个入口切入。

## 阶段

React更新时会分为两个阶段[render阶段与commit阶段](https://github.com/mbaxszy7/blog/issues/16)

### render阶段

在render阶段，React将更新应用于通过setState或render方法触发的组件，并确定需要在用户屏幕上做哪些更新--哪些节点需要插入，更新或删除，哪些组件需要调用其生命周期方法。最终的这些更新信息被保存在一个叫`effect list`的fiber 节点树上。当然，在首次渲染时，React不需要产生任何更新信息，而是会给每个从render方法返回的element生成一个fiber节点，最终生成一个fiber节点树， 后续的更新也是复用了这棵fiber树。

 render阶段被标记为纯的、没有副作用的，可能会被React暂停、终止或者重新执行。也就是说，React会根据产生的任务的优先级，安排任务的调度（schedule）。利用类似requestIdleCallback的原理在浏览器空闲阶段进行更新计算，而不会阻塞动画，事件等的执行。
 
 render阶段开始与`beginWork`，终止与`completeWork`

### commit阶段

在这个阶段时，React内部会有三个fiber树：

```text
current fiber tree: 在首次渲染时，React不需要产生任何更新信息，而是会给每个从render方法返回的element生成一个fiber节点，最终生成一个fiber节点树， 后续的更新也是复用了这棵fiber树。

workInProgress fiber tree: 
所有的更新计算工作都在workInProgress tree的fiber上执行。当React 遍历current fiber tree时，它为每个current fiber 创建一个替代（alternate）节点，这样的alternate节点构成了workInProgress tree

effect list fiber tree: workInProgress fiber tree 的子树，这个树的作用串联了标记具有更新的节点
```

commit阶段会遍历effect list，把所有更新都commit到DOM树上。具体的，首先会有一个pre-commit阶段，主要是执行getSnapshotBeforeUpdate方法，可以获取当前DOM的快照（snap）。然后给需要卸载的组件执行componentWillUnmount方法。接着会把current fiber tree 替换为workInProgress fiber tree。最后执行DOM的插入、更新和删除，给更新的组件执行componentDidUpdate，给插入的组件执行componentDidMount。

重点要注意的是，这一阶段是同步执行的，不能中止。


`commitRoot`方法是commit阶段工作的起点

#### commit阶段的三个流程

before mutation阶段（执行DOM操作前）

mutation阶段（执行DOM操作）

layout阶段（执行DOM操作后）

## React.createElement

## ReactDOM.render

## fiber

## Update

## Hook