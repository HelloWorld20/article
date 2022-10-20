---
title: Next.js重构博客笔记
date: 2022-09-16 19:28:03
tags: [笔记,Nextjs,tailwind]
---

学习了Next.js后，打算用Next.js重构一波自己的博客。Next.js的图片优化、字体优化，SEO、core web vital等特性非常适合博客，所以打算重构一下。网上搜了一圈，没有合适的模板，所以以官网的[blog-starter](https://github.com/vercel/next.js/tree/canary/examples/blog-starter)模板来实现。一是为了使博客达到最佳性能，二也是可以学学Next.js这个优秀的框架。

此偏文章是笔记的形式。主要是记录开发过程。可能会组织得比较乱。

# 正文

## 添加别名

分两个，webpack别名和typescript别名

```javascript
// next.config.js
module.exports = {
  webpack: (config) => {
    config.resolve.alias['@'] = path.resolve(__dirname)
    return config;
  }
}
```

```json
// tsconfig.json

{
	"baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
}

```


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


## 引入随机名言

旧版hexo模板的博客有个随机名言+打字机效果的UI，觉得挺炫酷的，也想加一个。

了解到数据来自一个叫[一言](https://developer.hitokoto.cn/)的开放服务，直接调用一个接口就会返回随机名言

打字机效果用的是[easy-typer-js](https://github.com/pengqiangsheng/easy-typer-js)库，安装后发现nextjs报错：`SyntaxError: Unexpected token 'export'``

原因是easy-typer-js包返回的esm，babel默认没有处理node_modules里的文件。

网上搜寻，nextjs的[issue](https://github.com/vercel/next.js/discussions/17685)里有人推荐使用[next-transpile-modules](https://github.com/martpie/next-transpile-modules)库来处理node_modules里的esm

跟目录新增next配置文件`next.config.js`，写入以下内容即可

```js
// next.config.js
const withTM = require('next-transpile-modules')(['easy-typer-js']); // pass the modules you would like to see transpiled

module.exports = withTM({});
```
## fetch

nextjs的fetch方法有点小奇怪。并没有专门的文档来介绍。根据其声明文件来看，fetch方法返回`Promise<Response>`对象。其对象包含请求头的信息，但是并不包含请求体。获取请求体则需要调用`res.json()`，奇怪的是res.json()返回的还是一个[promise](https://stackoverflow.com/questions/37555031/why-does-json-return-a-promise)。网上说，fetch().then()得到的是请求头。fetch().then(res => res.json()).then(res => console.log(‘此处才是请求体’))，需要then两次。

所以获取名言接口+useSWR的写法是

```jsx

function fetcher() {
  return fetch('https://v1.hitokoto.cn/').then(res => res.json())
}

const { data } = useSWR('1', fetcher);

```
## 引入photoswipe，图片预览插件

这次打算实现相册的功能，所以打算引入保存在github star里许久的photoswipe库。首先拿一个文章详情页面做实验

看了文档，发现photoswipe需要的html结构是要可以点击放大的图片的父组件必须是`a`标签，且a标签的href或者data-pswp-src必须填入图片的url。

当然，remark转出来的html是不会给图片包上一层a标签的，想要达到photoswipe要求的结构，则需要修改remark的输出结果。

经过大量的搜寻，remark有[大量的插件](https://github.com/remarkjs/remark/blob/main/doc/plugins.md#list-of-plugins)可以增强markdown转html的功能。发现[remark-image](https://github.com/remarkjs/remark-images)就可以给img标签套上一层a标签，但是它只管给a标签的href属性填为图片链接。还需要定制一下a标签的属性。

（这里走了个弯路，当时认为a标签必须设置`data-pswp-src`参数，但是后来发现直接设置`href`也是可以的，remark-image已经可以实现photoswipe的基本结构，但是要增强功能，还是改属性的，弯路也没白走）

修改remark转html的代码，新增一个插件
```javascript
import { remark } from 'remark'
import html from 'remark-html'
import remarkImages from 'remark-images'

export default async function markdownToHtml(markdown: string) {
  const result = await remark().use(html).use(remarkImages).process(markdown)
  return result.toString()
}
```
插件中看到[remark-attr](https://github.com/arobase-che/remark-attr),可以定制元素属性，这就是我想要的功能。但是可惜的是，该插件已经[不兼容最新的 remark，且不再维护](https://github.com/arobase-che/remark-attr/issues/34#issuecomment-1229378243)

又经过大量搜寻和查看源码，发现remark-html[内部](https://github.com/remarkjs/remark-html/blob/main/index.js#L43)是先把mast（markdown ast）转换成hast（html ast），操作后再把hast转换成html。那么我也可以操作hast来实现修改a标签的属性。

先把代码copy到本地。看到插件中用到一个专门[处理ast的库](https://github.com/syntax-tree)，其中[unist-util-map](https://github.com/syntax-tree/unist-util-map)是可以遍历处理hast的方法。所以在中间插入一段代码：

```javascript
cleanHast = map(cleanHast, (node) => {
	if (isMarkedImage(node)) {
	  node.properties['data-pswp-src'] = node.properties.href
	}
	return node;
});
```
找到插入的a标签修改其属性即可。

（又走了一个弯路，remark-images不是给图片包一层a标签，而是给图片url的plain text包一层a标签，并且给当前的图片url变成图片标签）

改造一下源码的判断条件即可。

```javascript

// visitParents(tree, 'text', (node, parents) => {
visitParents(tree, 'image', (node, parents) => {

	//  const value = String(node.value).trim()
    const url = node.url;
    //  if ((isUrl(value) || isImgPath(value)) && isImgExt(value)) {


	const image = {
       type: 'image',
       // url: value,
       url,
       alt: '',
       position: node.position
     }
	 /** @type {Image|Link} */
	 let next = image
	 // Add a link if we’re not already in one.
	 if (!interactive) {
	   next = {
		 type: 'link',
		 // url: value,
		 url,
		 title: 'flag',
		 children: [image],
		 position: node.position
	   }
	 }

})

```


## 更强的lightgalleryjs

photoswipe另一个必填项是，必须要指定图片宽高，这点就真的有点为难了。实现起来估计得在运行时计算宽高在动态设置html的markup。因为博客内容都是markdown转换生产的html字符串，在Nextjs里实现起来，实在不太优雅。

所以又搜寻了一番，发现另一个库更符合实际：[lightgallaryjs](https://www.lightgalleryjs.com/)

该插件可以插件式的引入功能，非常灵活的自定义html的markup，而且还自带React组件，都省的自己封装一次。

只需要在remark-html里的hast给合适的node添加指定class，

```javascript
cleanHast = map(cleanHast, (node) => {
      if (isMarkedImage(node)) {
        node.properties.class = GALLERY_ITEM_CLASS
      }
      return node;
    });
```

然后再设置给lightgallaryjs即可

```jsx

import LightGallery from 'lightgallery/react';

// Plugins
import lgThumbnail from 'lightgallery/plugins/thumbnail'
import lgZoom from 'lightgallery/plugins/zoom'

import 'lightgallery/css/lightgallery.css';
import 'lightgallery/css/lg-zoom.css';
import 'lightgallery/css/lg-thumbnail.css';


<LightGallery speed={500} plugins={[lgThumbnail, lgZoom]} selector={`.${GALLERY_ITEM_CLASS}`}>
	<div>...</div>
</LightGallary>

```

是时候淘汰老旧的Photoswipe了


## 子路由刷新404

next.config.js里添加配置：`trailingSlash: true`

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

## window is not defined

因为部分代码是会再服务端执行的，所以在服务端执行的代码不能执行window等仅在浏览器环境才能访问的对象。

而如果要访问，那window应该在componentDidMount或者useEffect里操作。

会在服务端执行的代码除了getStaticProps,getStaticPaths等还有constructor、componentWillMount、getDerivedStateFromProps、render。所以不要在这些生命周期、函数内调用仅浏览器端能访问的对象。