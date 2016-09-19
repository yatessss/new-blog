---
layout: post
title: 《JavaScript高级程序设计》读书笔记
date: 2016-04-18
tags: ['JavaScript']
categories: JavaScript	
---

最近在复习《JavaScript高级程序设计》，然后记一些自己原来不懂的地方吧，不会整个总结，就是按知识点一个一个把新学到的列出来。

1. JavaScript = ECMAScript + DOM + BOM

2. 在html中可以用`<noscript>`的标签，在这个标签内写的html内容，会在浏览器不支持脚本或者浏览器禁用脚本的时候显示出来。

3.  JavaScript的变量是区分大小写的。

4. isFinite()函数可以确定一个数值是不是有穷的即在不在浏览器支持的最大值和最小值之间，如果是就会返回`true`。

5. Number()􏰟parseInt()􏰠和parseFloat()，第一个可以把任何类型转换为数值，另外两个可以把字符串转换为数值。

6. 转换为字符串有两种方式：toString()和String()。toString()方法可带参数表示输出数值的基数，但不能转换undefined和null；String()方法的规则是：有toString()方法调用toString()，如果是undefined和null则返回相应字符串。

7. 每个创建的对象实例都有下列方法和属性：constructor，hasOwnProperty，isPrototypeOf，propertyIsEnumerable，toLocaleString()，toString()，valueOf()。

8. 复制基本类型的变量值得时候会复制出一个副本（互相不会影响），而复制一个对象类型的时候会添加一个指针而已（修改一个会影响另一个）。

9. 确定一个值是那种基本类型用`typeof`，确定值是哪一种引用类型用`instanceof`。

10. 通过`[]`来访问对象属性的时候，`[]`中可以是变量，也可以是非字母非数字的属性名。

11. arguments.callee是一个指向正在执行函数的指针，在递归当中可以应用。（在严格模式中这个不可以用，可以用一个命名函数表达式，`var a = (function b (){})`）在函数中调用`b`这个函数名。

12. 使用`var`定义在全局的变量其实是window下的一个属性，他的Configurable属性设为了false（不可配置的），所以不能用delete删除掉。
13. 每个框架有自己的window，top对象指向对外层框架，也就是浏览器窗口。
14. BOM中window.moveBy、window.moveTo、window.resizeTo、window.resizeBy这些方法在浏览器默认是禁用的，且只作用于最外层窗口对象。
15. 浏览器有三种提示框alert()（只有确认键）、confirm()（有确认和取消键）和prompt()（除了确认取消还有输入框）。
16. 一个页面的两个框架如果是不同子域将无法通信，可以将document.domain设置为同一个主域名，他们之间就可以互相通信了。（有限制，只可以从紧绷`p2p.wrox.com`向松散`wrox.com`改变，反之不可）。
17. 