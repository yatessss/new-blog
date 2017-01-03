---
layout: post
title: 重排reflow与重绘repaint
date: 2016-04-29
tags: ['JavaScript']
categories: JavaScript	
---

### 重绘与重排

最近看了一本书《高性能JavaScript》，里面有很多有用的代码片段，后面我准备把一些摘抄记录下来。

今天主要想说说重绘与重排，这个也是《高性能JavaScript》中讲的一部分，我来总结一下吧。

首先先要了解一下浏览器渲染页面的过程，当浏览器下载完页面所需要的html、css、js和图片等后就会开始解析并生成两个内部的数据结构：DOM树和渲染树。
DOM树是表示页面的结构，渲染树是表示DOM节点如何显示。

DOM树不必多说，渲染树会结合DOM树和DOM节点对应的CSS样式去理解页面上每个元素的样式（比如内外边距，边框，位置等），去构建一个渲染树。渲染树完成后浏览器就开始绘制（paint）页面元素。

关于这部分可以看这个链接[渲染树构建、布局及绘制
](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=zh-cn)

当DOM树的结构和CSS产生变化影响到页面布局或元素的几何属性的时候，浏览器会重新构建渲染树，重新构建渲染树的过程就叫做`重排（reflow）`，重排完成后，浏览器会重新绘制受到影响的部分到屏幕中，这个过程叫做`重绘（repaint）`。

并不是所有的变化都会影响到页面布局和元素的几何属性（即发生重排），下面的情况会发生重排：

* 添加和删除可见的DOM元素。
* 元素位置改变。
* 元素尺寸改变（内外边距，边框厚度，宽高等）。
* 内容改变（文本改变，图片尺寸改变）。
* 浏览器窗口尺寸变化。
* 页面渲染器初始化。

不改变页面布局和元素几何属性的变化（比如背景色变化），只会发生一次重绘，并不会发生重排。

### 减少重绘与重排

因为重绘和重排需要大量计算，会影响页面的响应速度，所以我们应该尽量减少和避免重绘和重排。

#### 改变样式

看下面这段代码：

```js
var el = document.getElementById('mydiv');
el.style.borderLeft = '1px'; 
el.style.borderRight = '2px';
el.style.padding = '5px';
```

这样添加样式，每一次都会改变元素的几何属性，在一些旧版浏览器中可能会引起三次重排（现代浏览器会做优化处理，发生一次重排），所以可以优化一下代码，合并样式一次性修改：

```js
var el = document.getElementById('mydiv');
//替换样式
el.style.cssText = 'border-left: 1px; border-right: 2px; padding: 5px;';

//保留原有样式
el.style.cssText += '; border-left: 1px;';

```
这样修改只会引起一次重排，更为高效。还有一种做法就是为想要修改的部分添加一个`class`使用css一次性修改。

#### 批量修改DOM

如果我们需要对DOM进行一系列操作的时候，可以通过下面的做法来减少重绘和重排：

1. 使元素脱离文档流。
2. 对其进行操作。
3. 把元素带回文档中。

这样如果我们在第二步进行多次操作时，也只会在第一步和第三步触发两次重排。

有三种基本方法可以使DOM脱离文档流:

1. 隐藏元素，修改，重新显示。
2. 使用文档片断（document fragment），在当前DOM外构建一个子树，再把它插入文档中。
3. 把原始元素拷贝到脱离文档的节点中，修改后在把原始元素替换掉。

用代码来说明三种方法：

比如现在我们有一个`ul`列表，我们用一个方法`appendDataElement()`往列表中添加`li`。

```js
//要操作的列表
var ul = document.getElementById('mylist');
//向列表中添加li，data是li中的内容
appendDataToElement(ul, data);
```

如果我们不使用任何方法的话，每插入一个`li`就会触发一次重排，这样是很影响性能的。所以我们可以使用上面的三种方法。

方法一：

```js
var ul = document.getElementById('mylist');
ul.style.display = 'none'; 
appendDataToElement(ul, data);
ul.style.display = 'block';
```

方法二：

```js
var fragment = document.createDocumentFragment();
appendDataToElement(fragment, data);
document.getElementById('mylist').appendChild(fragment);
```

方法三：

```js
var old = document.getElementById('mylist'); 
var clone = old.cloneNode(true);
appendDataToElement(clone, data);
old.parentNode.replaceChild(clone, old); 

```

文章中是推荐我们使用第二种方法，因为这种方法本来设计的初衷就是为了解决这类任务的（更新和移动节点）。而且这种方法只触发一次重拍，只访问一次DOM节点。


既然说到了重绘和重排，顺便就说一下动画，在动画中我们可以尽量使用`transform`和`opacity`，因为他们会不会触发重绘。具体可以看看这两篇文章：

1. [CSS动画之硬件加速](http://www.w3cplus.com/css3/introduction-to-hardware-acceleration-css-animations.html)
2. [CSS动画的性能优化](http://zencode.in/14.CSS%E5%8A%A8%E7%94%BB%E7%9A%84%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.html)