---
title: 记一次升级monorepo的经历
date:  2020-11-25 22:21:19
categories: 
    - 总结,lerna
tags: 
    - 总结,lerna
---

由我主要负责的一个项目，最近需要另外一个项目组的人做定制开发。当然了，定制开发多多少少会影响原有的代码。然后他们项目技术负责人提到：能不能参考另外一个项目，“分包”开发。然后我醍醐灌顶，突然明白，最近公司里诞生的新架构的目的与作用。

最早是数月前，我们组大牛带的团队搞的新项目分享会提到的一个新技术名词：lerna。从第一次听到lerna到现在，我都不太清楚，lerna是什么，有什么作用。而且，另外一个与我相关的项目，也有大佬进行改造，我还一直不明白，为什么要把项目改得更复杂。这次终于明白。。。

lerna，官方说明是`多包存储管理器（monorepo）`，起因是，在多个项目依赖开发时，比如project-1依赖project-2，得用npm link手动引入依赖，然后才能被project-1使用，发布时还得经过复杂的打包、发布、打tag、打版本号。项目少则手动无所谓，但是依赖多起来，几乎不可能维护。lerna则可以让project-1与project-2合在一个项目里，并且每个project都可以独立发布，但是开发过程中可以时时更新，依赖。

lerna用于我们项目中的话，可以把公共组件、公共方法、配置抽离成各个project、然后不同需求的业务代码各成立一个project，这么一来，各个业务代码就完全独立开来，完全不冲突，完全不用担心影响其他业务代码。非常完美。

# lerna概念

lerna需要全局安装

	npm install lerna -g
	
然后初始化项目

	lerna init
	
那么就会得到一个这样的文件结构

	├─packages
	│  ├─project1
	│  ├─project2
	│  └─project3
	├─package.json
	└─lerna.config.json

所有项目都在packages文件夹下，每个project就是一个独立的项目。然后需要初始化每个项目

	cd packages/project1
	npm init -y

就会创建package.json
然后手动修改package.json的name参数，命名为独一无二的名称，以后每个包都得以这个名称为主

检查一下包，有没有初始化成功

	lerna list
	
列出所有初始化成功的包

把代码迁移到每个project中。

然后，安装依赖。

	lerna bootstrap --hoist

包之间互相依赖关联

	lerna add projectName1 --scope=pojectName2
	
这样，project2就加入了project1的依赖

到这基本上lerna就差不多了。

## lerna常用命令

### init

初始化项目，生成基本文件结构，有两种，fixed与indepancent。

默认是fixed。indepancent需加`--indepancent`

我的情况用fixed。因为不打算每个project单独发布。所有代码都统一发布上传就好

### list

列出所有包

### boostrap

最重要的命令。会把所有包的依赖都安装，包括包与包之间的依赖。是本地包的话，会在project的node_modules下创建一个软连接，这样就能直接引用了。

#### --hoist参数

hoist参数可以让各个包公用到的依赖抽离到跟目录安装，避免重复安装。且只能这样，才能让各个包互相依赖。

### add

packages间引用就不能npm install了，需要用add命令添加

	lerna add packages-1 --scope=packages-2
	
就能给packages-2加上packages-1当作依赖。

### clean

lerna项目可能会在各个packages安装node_modules。--hoist模式还会在跟目录安装各个packasges公共的node_modules。lerna提供clean命令，一键清除所有node_modules

 lerna clean
 

# 还需修改

## 手动symlink

lerna只能把包软链到node_modules里，这样则不能watch其他项目的源码变化。

----------更新------------

项目是基于Taro3的，默认是可以监听到node_modules的软链改变，所以不是问题。默认还是基于lerna建立的软链来开发就好


## 手动symlink兼容

亲测，只能用shelljs的shell.ln命令来进行软链，ln命令在windows下和Ubuntu下都要问题。。。nodejs的fs.symlink方法也不行。。。

## 循环引用的问题



## 全局eslint

## typescript的声明文件的问题


