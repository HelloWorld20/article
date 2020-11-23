---
title: 关于React源码学习（草稿）
date: 2020-11-21 17:05:24
categories: 
    - 总结,React
tags: 
    - 总结,React
---

最近尝试学习React16的源码，学习阶段，先建个草稿，记录片段。


[React Fiber架构-司徒正美](https://zhuanlan.zhihu.com/p/37095662)
React16则是需要将虚拟DOM转换为Fiber节点，首先它规定一个时间段内，然后在这个时间段能转换多少个FiberNode，就更新多少个。

因此我们需要将我们的更新逻辑分成两个阶段，第一个阶段是将虚拟DOM转换成Fiber, Fiber转换成组件实例或真实DOM（不插入DOM树，插入DOM树会reflow）。Fiber转换成后两者明显会耗时，需要计算还剩下多少时间。