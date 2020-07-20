---
title: 从零配置webpack4
date: 2018-10-09 09:59:16
categories: 
	- webpack
    - 技术总结
tags: 
    - webpack
    - 技术总结
---

有机会搭建一个全新的项目，类似移动端门户网站的那种。然后Vue-cli各种问题。反正有时间，所以想从头搭建一个，刚好可以学习一下。

<!-- more -->

# 计划

第一步肯定要好好想想，技术的选型。

* Vue + Vue-router + axios + Vuex + webpack4
* sass
* mint UI
* eslint
* 单元测试（次要）

没有选择Typescript

关于webpack，需要

* js混淆压缩，去除死代码
* js公共代码分离打包
* sass转换、css分离、合并、压缩
* 打包分析
* 支持vue单文件
* 静态资源

项目中要支持

* 路由拦截
* 接口封装
* 

# 步骤

## 建目录结构

## 建webpack配置

新建两个webpack配置文件，一个用于本地调试（webpack.dev.js），另一个打包上线（webpack.pro.js）

写入基本的配置：entry、output、module、plugins；

### dev

dev模式只需要最基本的配置:copy-webpack-plugin、html-webpack-plugin、vue-loader、sass-loader等必须的功能。为了打包速度，其余代码压缩、css抽离等都不需要；

dev额外的需要配置[webpack-dev-server](https://www.webpackjs.com/configuration/dev-server/)，然后再package.json文件里script字段里加上一句：`webpack-dev-server --config build/webpack.dev.js`就能跑起来了。

dev还要一个额外的配置：[devtool](https://www.webpackjs.com/configuration/devtool/)方便开发调试时可以看到源码。

### pro

所有优化的项目都该下上去

package.json script里加上`"build": "webpack --config build/webpack.pro.js"`

安装webpack、webpack-cli和其他相关依赖即可

## 关于loader

## 样式loader

关于css的几个loader，官网的注释是：

	"style-loader", // 将 JS 字符串生成为 style 节点
    "css-loader", // 将 CSS 转化成 CommonJS 模块
	"sass-loader", // 将 Sass 编译成 CSS，默认使用 Node Sass

没什么特别的。注意顺序就好。如果处理的是.css，则去掉sass-loader

安装 style-loader、css-loader、sass-loader；

### 抽离公共css

生产环境需要抽离css，所以用到`mini-css-extract-plugin`（webpack旧版本用的是extract-text-plugin），而`mini-css-extract-plugin`与`style-loader`是冲突的，所以生产版本的css loader配置应该是：

	MiniCssExtractPlugin.loader,
	// "style-loader", // 将 JS 字符串生成为 style 节点
	"css-loader", // 将 CSS 转化成 CommonJS 模块
	"sass-loader", // 将 Sass 编译成 CSS，默认使用 Node Sass


### postcss-loader

postcss-loader是功能比较强大的css预处理loader，但是我暂时只需要它的autoprefix前缀功能。

生产环境下，在`css-loader`前加上`postcss-loader`。

然后在项目根目录新建文件：`postcss.config.js`。里面可以细粒化的配置postcss。

	module.exports = data => ({
		// parser: this.webpack.resourcePath.endsWith('.css') ? 'sugarss' : false,
		// 这里应该是“sugarss”，但是sugarss不能处理sass，所以不能写
		parser: false,
		plugins: {
			'postcss-cssnext': {},
			'autoprefixer': {},
			'cssnano': data.env === 'production' ? {} : false
		}
	});

	
然后根据报错依次安装sugarss、postcss-cssnext、autoprefixer、cssnano即可。

**postcss-loader是个大项，可以好好学学**

## babel-loader

babel-loader是把js的ES6以上的语法转换为ES5语法的loader，为了前端的浏览器兼容性，还有可以在开发的时候大胆的用上新语法，babel还是必不可少的。

虽然在网上好多文章都要给loader写很多配置，但是目前我构建的（babel-loader版本8.0.5）来看，不需要任何额外的配置（只填了`cacheDirectory: true`）。只要webpack中指定了babel-loader，则babel会自动找到`.babelrc`文件，读取其配置。在babel的配置文件里配置babel即可。

```
{
	"presets": ["@babel/preset-env"],
	"plugins": ["@babel/plugin-syntax-dynamic-import"]	// 支持动态路由 import('')语法
}
```
可能是最新版内置了很多默认配置吧

然后根据需求、报错安装与配置指定配置即可

## vue-loader

vue开发中绝对少不了vue单文件。而vue-loader就是把`.vue`文件转换为js的loader。同时还包括自动处理资源路径、scoped css功能。

在这次安装的`14.2.2`版本的配置貌似也是很自动化的，会根据`lang`属性自动识别其他已经配置的loader而自动转换。（\<template\>应该会自动走vue-template-compiler，script，应该会匹配.js后缀的loader）

新版本`15.0.0`之后的版本貌似就不会那么智能了，需要手动配置很多东西。略。

## 关于plugins

## 关于optimization

# jest 单元测试

[https://vue-test-utils.vuejs.org/zh/guides/testing-single-file-components-with-jest.html](https://vue-test-utils.vuejs.org/zh/guides/testing-single-file-components-with-jest.html)

安装jest、@vue/test-units、vue-jest
