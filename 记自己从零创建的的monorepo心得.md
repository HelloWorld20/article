---
title: 记自己从零创建的的monorepo心得
date: 2021-03-10 14:13:48
categories: 
    - 总结,yarn workspace,monorepo
tags: 
    - 总结,yarn workspace,monorepo
---

# 前言

在工作中遇到了用lerna和yarn workspace为基础创建的monorepo项目，而且也有幸研究了一番他们的代码并且在自己的工作项目中改造了一番。虽然最终放弃应用在项目中，但是觉得monorepo可以利用在我个人的项目中。把我个人的前端项目、后端服务、工具函数、工具类项目都整合到一个monorepo中。最大化的复用代码，模块化代码。

# yarn workspace与lerna

我对lerna与yarn workspace的探索路径是这样的。一开始，是从公司的项目和公司大佬的介绍中，了解到monorepo的概念和lerna这个工具。于是乎去Google一番、也了解到monorepo的概念与优缺点。并且在工作的项目中进行了一番改造。

但是在进行过程中，遇到不少问题。比如

* lerna bootstrap（安装、关联依赖）太慢。
* 给某个模块安装依赖、太麻烦。。语法与npm、yarn不一样、还得去搜，还搜不到
* 相关文档还是略少
* 等等

然后过了一段时间、另一位大佬分享了自己的monorepo项目，从lerna换成了yarn workspace，说是不需要上传npm，其实用不到lerna

....

好吧，之前直接就是 lerna 加上默认的npm，没去了解lerna + yarn workspace。

然后现在再返过来去了解一下yarn workspace

发现，其实yarn workspace就满足我的需求（除了上传到npm），而且实现的更好（毕竟之前用过yarn）

所以最后，用的yarn workspace去组织我个人的项目代码。

# 正文

首先考虑下如何组织项目，经过多番考虑与修改，可以把包分类3类

* 前端项目：以template-\*命名，以webpack与webpack-dev-server为中心的React项目
* 服务端项目：以server-\*命名，提供RESTful的node（express.js）服务
* 工具类项目：以tools-\*命名，一些提供cli功能的项目，如webpack打包、静态文件上传cdn等工具
* 还有个utils：存放工具函数





