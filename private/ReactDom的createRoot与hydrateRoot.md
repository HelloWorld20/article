---
title: ReactDom的createRoot与hydrateRoot
date: 2022-10-14 15:14:00
tags: [总结,React]
---


最大的区别再onLoad之后

createRoot:
![[images/Pasted image 20221014151622.png]]


hydrateRoot:
![[images/Pasted image 20221014151709.png]]
hydrateRoot前面多了很多段细小的运行，Compile Code

hydrateRoot与createRoot其实并没有太多区别。只是hydrate尽量的复用服务端传来的dom节点而已。如果节点不一样，是会报错的。

可以做个实验，再调用hydrate之前，操作一下dom，就会报如下错误：

*Uncaught Error: Hydration failed because the initial UI does not match what was rendered on the server.*

[来源文章](https://juejin.cn/post/7122111668053606437)