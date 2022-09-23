---
title: Next.js重构博客笔记
date: 2022-09-16 19:28:03
tags: [笔记,Nextjs,tailwind]
---

学习了Next.js后，打算用Next.js重构一波自己的博客。Next.js的图片优化、字体优化，SEO、core web vital等特性非常适合博客，所以打算重构一下。网上搜了一圈，没有合适的模板，所以以官网的[blog-starter](https://github.com/vercel/next.js/tree/canary/examples/blog-starter)模板来实现。一是为了使博客达到最佳性能，二也是可以学学Next.js这个优秀的框架。

此偏文章是笔记的形式。主要是记录开发过程。可能会组织得比较乱。

# 正文

## 引入代码高亮

Nextjs的模板中用的remark没有自带代码高亮。网上搜寻后了解到hightlight.js是专业的代码高亮库。

	npm install hightlight.js


使用方法很简单

在需要的页面引入资源然后初始化即可，代码高亮一般只有在详情页有，在详情页加上即可

```jsx
import "highlight.js/styles/vs2015.css"; // 样式文件
import hljs from "highlight.js/lib/common"; // highlight.js核心
import javascript from "highlight.js/lib/languages/javascript"; // 单独使用js部分
import xml from 'highlight.js/lib/languages/xml';


useEffect(() => {
    hljs.registerLanguage("jsx", javascript);
    hljs.registerLanguage("xml", xml);
    hljs.highlightAll();  // 高亮所有'pre code'
}, [])

```
一个一个引入太麻烦，highlight内置了一组语言

```jsx
import "highlight.js/styles/vs2015.css"; // 样式文件
import hljs from "highlight.js/lib/common"; // highlight.js核心
import javascript from "highlight.js/lib/languages/javascript"; // 单独使用js部分
import xml from 'highlight.js/lib/languages/xml';


useEffect(() => {
    hljs.highlightAll();  // 高亮所有'pre code'
}, [])

```
调用highlightAll方法后，会自动给所有\<pre\>\<code\>\</code\>\<pre\>标签内的代码加上高亮。

还好remark会把代码生成为\<pre\>\<code\>\</code\>\<pre\>格式，这点就不需要修改了。

最后再封装成一个hooks，需要的页面一个`useHightLight()`即可


# todo

* 主页、文章详情、归档、标签、关于页面。
* 文章搜索功能
* 相册
* 评论、访问量
* 文章图片接入PhotoSwipe点击放大

# bug log

## 报错("[object Date]") cannot be serialized as JSON

Error: Error serializing `.allPosts[0].date` returned from `getStaticProps` in "/". Reason: `object` ("[object Date]") cannot be serialized as JSON. Please only return JSON serializable data types.

原因是Date类型的字段不能作为getStaticProps/getServerSideProps的返回值返回到前端。因为官方觉得这会对UX测试带来困难。

解决办法是把Date类型toString转成字符串即可

这个问题只会在开发模式产生。

来源：

[github issue](https://github.com/vercel/next.js/issues/11993#issuecomment-617937409)
