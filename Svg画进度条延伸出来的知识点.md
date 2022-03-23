---
title: svg画进度条需求复盘
date: 2022-03-22 23:12:48
tags: [svg,总结]
---

最近的一个需求是自定义加载页，我负责的是C端的进度条的实现。经过讨论，进度条应该用SVG实现，有直线、圆环两种。还有两个组件要实现进度跟随。也就是两个组件要跟着进度移动。最终实现起来，还是有不少知识点可以记一下的。一是SVG是个盲点，二是这个需求破天荒的用了不少数学。。。


# 正文

# 关于svg

## svg

\<svg />标签是svg的跟标签，必须要有。没有则不会渲染。其中有一个特有的属性：`viewBox`，用于定义视图范围。也就相当于定义一个`二维坐标系`，svg上的内容都是根据坐标点来画的，超过这个坐标系的内容不可见。

一般还要设置svg标签的width、height属性。这个则是svg占用的范围。以像素为单位。

```html
<svg width="800" hight="800" class="svg" viewBox="0 0 400 400" fill="none">
	 <path
		class="dashed path1"
		d="M360,200A160,160,0,1,1,200,40,160,160,0,0,1,360,200Z"
		style="stroke-linecap: round"
		stroke="green"
		stroke-width="12"
	 />
	 <path
		class="dashed path2"
		d="M 4 15 H 98"
		stroke="red"
		stroke-width="12"
	 />
 </svg>

```

所以一般svg的数据上的位置参数不一定与真实像素一致。只有viewBox与width、height一样宽高时，位置参数才与真实像素一致。

后面也会说到，实际开发中，想要精确的控制尺寸。需要进行转换。

关于其他属性，如style、stroke、stroke-width等。则是可以让其子元素生效。如果多个子元素有相同的属性，可以给\<svg/>标签设置，而不是每个标签都设置。

## path

真正描述内容的是svg标签的子元素。有非常多类型。而目前用到的，也是功能最强的是就是\<path/>标签。\<path/>应该是可以描述任意形状的内容。




### 属性d
\<path/>强大的原因就是d参数。内容太多，则不展开讲。可以参考：[svg之path详解](https://www.jianshu.com/p/c819ae16d29b)

d属性的格式是以`一个字母加上一连串数字组成一个命令`。字母代表动作，数字代表该动作的参数。字母分大小写。大写表示绝对定位，小写表示相对定位。

以`M 4 15 H 98`为例，`M`代表`移动到`，`M 4 15`意思是移动到(4,15)这个坐标点。`H`代表`横向画线`，`H 98`意思是横向画一条到98的线。

所有动作摘抄如下。篇幅有限，暂只记录那么多。

M = moveto  
L = lineto  
H = horizontal lineto  
V = vertical lineto  
C = curveto  
S = smooth curveto  
Q = quadratic Bézier curve  
T = smooth quadratic Bézier curveto  
A = elliptical Arc  
Z = closepath

## 一些常见的属性于方法

### getTotalLength
	SVGGeometryElement.getTotalLength(): number

仅对path元素有效，可以返回path的总长度。也是线段的长度。

对于例子的`.path2`，值为94，因为线段是从4到98

### getPointAtLength

	SVGGeometryElement.getPointAtLength(distance: number): DOMPoint
	
	

### stroke-width

### fill

### stroke

### stroke-dasharray

### stroke-dashoffset

### getBBox

### vector-effect="non-scaling-stroke"

 [stackoverflow-path.getTotalLength() returning wrong values](https://stackoverflow.com/questions/40297356/path-gettotallength-returning-wrong-values)


## 几种实现进度的方法

### 动态计算stroke-dasharray

### 动态计算stroke-dashoffset

### 动态计算d参数

## tweenjs

## 缓动函数

## 路径跟随

### 路径跟随+偏移

### 路径跟随+旋转



