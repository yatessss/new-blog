---
layout: post
title: 《你不知道的JavaScript》读书笔记——闭包
date: 2016-03-29
tags: ['JavaScript','闭包']
categories: JavaScript	
---

### 理解闭包

闭包是什么，怎么理解？

闭包就是一个作用域的外部变量保持着对这个作用域的引用，这个引用就叫做闭包。

这么讲有些晦涩，看这个代码：

```js
function foo() {
     var a = 2;
     function bar() {
         console.log( a );
     }
     return bar; 
} 
var baz = foo();
baz(); // 2 这就是闭包
```

因为`baz`本身是`foo`作用域之外的变量，根据作用域的规则，`baz`本身是不可以访问到`foo`作用域内的变量的。

本来在通常的情况下，在函数执行完后，如果函数在后面不再使用的时候，会进行垃圾回收机制，把`foo`函数内的作用域销毁，把不再使用到的内存释放掉。

但是正因为闭包，这个作用域没有被销毁。原因是`foo()`函数执行之后的返回值`bar`，就是`bar`内部的函数即`function bar(){console.log( a );}`，把这个赋值给了`baz`，`baz`在被调用时（`baz()`）因为`baz`中用到了变量`a`，而变量`a`是在`foo`的作用域中，所以`baz`必须得拥有`foo`函数的作用域闭包才能够正常运行，所以`foo`的作用域不会被销毁会一直存在，以便`baz`之后调用的时候能正常运行。

这么说还是很绕，简单讲就是`foo`函数外面的变量`baz`要用到`foo`作用域里面的东西，这就叫`baz`拥有`foo`的闭包。

**再简单讲就是函数调函数。（一个同事说的，一想好像有一些道理）**

***在定时器、时间监听器、Ajax请求、跨窗口通信、或者任何其他的异步（或者同步）任务中，只要是用了回调函数，实际上就是在使用闭包。***

### 循环和闭包

看一个循环的例子：

```js
for (var i=1; i<=5; i++) {
     setTimeout( function timer() {
         console.log( i );
     }, i*1000 );
} 
```

本来这段代码的预期是，分别输出1~5，每秒一次，每次一个。但是实际上他会每秒一个的频率输出五次6。

造成这样的原因，书上讲的是循环中的五个函数是在各个迭代中分别定义的，但是他们都被封闭在一个共享的全局作用域当中，实际只有一个`i`。

解决办法是运用IIFE，并在每次循环中的IIFE内保存`i`的值。

```js
for (var i=1; i<=5; i++) {
     (function() {
         var j = i;
         setTimeout( function timer() {
             console.log( j );
         }, j*1000 );
     })(); 
} 
```

再改进一下代码：

```js
for (var i=1; i<=5; i++) {
     (function(j) {
         setTimeout( function timer() {
             console.log( j );
         }, j*1000 );
     })( i );
} 
```

如果运用ES6就会更加简单，只要运用`let`声明的块作用域：

```js
for (let i=1; i<=5; i++) {
     setTimeout( function timer() {
         console.log( i );
     }, i*1000 );
} 
```

### 模块

模块也是利用闭包来实现的。

```js
function CoolModule() {
     var something = "cool";
     var another = [1, 2, 3];
     function doSomething() {
         console.log( something );
     } 
     function doAnother() {
         console.log( another.join( " ! " ) );
     } 
     return {
         doSomething: doSomething,
         doAnother: doAnother
     }; 
} 
var foo = CoolModule();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

这样的模式就是最常见的模块的实现方法。返回值是一个包含内部的函数的对象。因为`CoolModule()`是一个函数，必须调用这个外部的函数后才能创建一个包含内部作用域的闭包。并且返回对象含有的是内部函数而不是内部变量的引用，内部变量是隐藏且私有的状态。

模块也是函数，也可以接受参数：

```js
function CoolModule(id) {
     function identify() {
         console.log( id );
     }
     return {
         identify: identify
     }; 
 } 
 var foo1 = CoolModule( "foo 1" );
 var foo2 = CoolModule( "foo 2" );
 foo1.identify(); // "foo 1"
 foo2.identify(); // "foo 2"
```

ES6为模块添加了语法的支持，ES6可以把文件当做模块来加载，但是要注意ES6的模块没有行内格式，就是每个模块必须在一个单独的文件中。

`bar.js`

```js
 function hello(who) {
     return "Let me introduce: " + who;
 }
 export hello;
```
 
`foo.js`

```js
//仅从bar模块导入hello()
import hello from "bar"; 
 var hungry = "hippo";
 function awesome() {
     console.log(
         hello( hungry ).toUpperCase()
     );
 }
 export awesome;
```

`baz.js`

```js
//导入完整的foo和bar模块
module foo from "foo"; 
module bar from "bar"; 
 console.log(
     bar.hello( "rhino" )
 ); // Let me introduce: rhino
 foo.awesome(); // LET ME INTRODUCE: HIPPO
```

`import`可以将一个模块中的一个或多个API引入到当前作用域，并分别绑定在一个变量上。`module`会将整个模块的API引入并绑定到一个变量上。`export`会将当前模块的变量或函数导出为公共的API。