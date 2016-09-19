---
layout: post
title: 《你不知道的JavaScript》读书笔记——this
date: 2016-03-31
tags: ['JavaScript','this']
categories: JavaScript	
---

### 对this的误解

学习`this`之前，应该知道两句话。

首先要先消除对`this`的误解，明白第一句话：***`this`既不指向函数自身也不指向函数的词法作用域。***这句话很重要，现在看一下证明这句话的例子。

***既不指向函数自身***

看下面的代码：

```js
function foo(num) {
    console.log( "foo: " + num );
    // 记录foo被调用的次数
    this.count++;
}
foo.count = 0;
var i; 
for (i=0; i<10; i++){
    if( i > 5 ){
        foo( i );
    }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9
// foo􏱦被调用了几次？
console.log( foo.count ); // 0 -- WTF? 
```

在`foo.count = 0`执行的时候，确实是向函数`foo`添加了一个属性`count`，但是`this.count`中的`this`并不是指向`foo`函数本身。

***也不指向函数的词法作用域***

看下面代码：

```js
function foo() {
     var a = 2;
     this.bar();
 }
 function bar() {
     console.log( this.a );
 }
 foo(); // ReferenceError: a is not defined
```

是不是觉得应该输出2？但是`this`并不指向`foo`的词法作用域，实际上`this`在任何情况都不指向函数的词法作用域，使用`this`不可能在词法作用域查询到什么。

*****

### this指向什么

第二句重要的话：***this实际上是在函数被调用时发生的绑定，它指向什么全完取决于函数被调用的位置。***

***它指向什么全完取决于函数被调用的位置。***

首先就是要找到函数被调用的位置，找位置要分析调用栈（执行当前函数的位置），我们要找的调用位置就是当前执行函数的前一个调用的位置。说着绕看代码就很明了了：

```js
function baz(){
    //当前的调用栈是：baz
    //因此，当前的调用位置是全局作用域
    
    console.log('baz');
    bar(); //<--bar的调用位置
}
function bar(){
    //当前的调用栈是：baz -> bar
    //因此，当前的调用位置在baz中
    
    console.log('bar');
    foo(); //<--foo的调用位置
}
function foo(){
    //当前的调用栈是：baz -> bar ->foo
    //因此，当前的调用位置在bar中
    
    console.log('foo'); 
}
baz(); //<--baz的调用位置
```

看完这个应该就会明白了，this指向的位置，就是在调用的位置。如果`this`在`foo`函数中，`foo`函数的调用位置是`bar`，那`foo`函数中的`this`就指向`bar`。

***this实际上是在函数被调用时发生的绑定***

找到了指向的位置，就要找this绑定在哪个对象上。`this`绑定会有四条规则，每一条的优先级是不同的。

四条规则分别是：默认绑定、隐式绑定、显式绑定、new绑定。

优先级是这样的：

1. 是否在new中调用？如果是，this就绑定的是新创建的对象。
2. 是否通过call、apply、bind调用（显式绑定）？如果是，this绑定的就是指定的对象。
3. 是否在某个上下文对象中调用（隐式绑定）？如果是，this绑定的是那个上下文对象。
4. 都不是的话，就是默认绑定。在严格模式下绑定到`undefined`，否则就是全局对象上。

下面介绍四中绑定：

#### 默认绑定

看下面这个代码：

```js
function foo() {
     console.log( this.a );
 }
var a = 2;
foo(); // 2
```

我们应该已经知道了，这个里的`this`会指向全局作用域。但原因是什么？是因为`foo()`是直接使用，没有在别的函数或对象内部被调用，所以就是默认绑定，默认绑定的`this`就会指向全局作用域。

但是如果在严格模式中，全局对象无法使用默认绑定，`this`就会绑定到`undefined`：

```js
function foo() {
    "use strict";
    console.log( this.a );
}
var a = 2;
foo(); // TypeError: this is undefined
```

#### 隐式绑定

如果`this`所在的函数被某个对象拥有或者包含，函数在运行时就会有这个对象的上下文，隐式绑定规则就会把函数中的`this`（隐式）绑定到这个上下文对象上。如果是一个链式调用呢，就只会关心最后一次调用时的上下文。

看下面代码：

```js
function foo() {
     console.log( this.a );
} 
var obj2 = {
    a: 42,
    foo: foo 
}; 
var obj1 = {
    a: 2,
    obj2: obj2 
}; 
obj1.obj2.foo(); // 42
```
像这样的链式调用呢，`this`最后在`obj2`中被引用，所以`this.a`其实就是`obj2.a`，就是42。

***隐式丢失***

```js
function foo() {
     console.log( this.a );
} 
var obj = {
    a: 2,
    foo: foo 
}; 
var bar = obj.foo; // 􏰃􏰄函数别名
var a = "oops, global"; //a是全局对象的属性
bar(); // "oops, global" 
```

因为`bar()`是一个不带任何修饰的函数调用，所以应用了默认绑定。

还有一个更常见的问题，在传入回调函数的时候：

```js
function foo() {
     console.log( this.a );
 }
 function doFoo(fn) {
     // fn是参数，其实引用的是foo
     fn(); // <-- 调用位置
} 
 var obj = {
     a: 2,
foo: foo }; 
var a = "oops, global"; // a是全局对象属性
doFoo( obj.foo ); // "oops, global" 
```
参数传递其实就是一种隐式赋值，所以我们传入的函数时也会被隐式赋值，隐式赋值会在全局作用域创建一个变量，所以结果就和上个例子一样。

语言的一些内置函数的本质其实也是传递参数，所以也会隐式赋值，就是出现绑定的丢失。

比如`setTimeout`函数：

```js
function foo() {
     console.log( this.a );
} 
var obj = {
    a: 2,
    foo: foo 
}; 
var a = "oops, global"; // a是全局对象属性
setTimeout( obj.foo, 100 ); // "oops, global" 
```

JavaScript内部`setTimeout`的实现，会传递参数：

```js
//类似实现
function setTimeout(fn,delay) {
    fn(); 
} 
```

#### 显式绑定

JavaScript提供了`call()`和`apply()`方法，可以直接指定this的绑定对象，称之为显式绑定。就像下面这样：

```js
function foo() {
    console.log( this.a );
} 
var obj = { 
    a:2 
};
foo.call( obj ); // 2
```
这样通过`foo.call()`，可以把`this`绑定到`obj`上面。

如果在`call()`中传入一个基本类型来当做`this`的绑定对象，那基本类型会调用基本包装类，把它变成对象的形式。

但显示绑定还是不能解决绑定丢失的问题，显式绑定的一个变种可以解决这个问题。办法就是在显式绑定的外面再加一个包裹函数，负责接收参数并返回值，这种方法叫***硬绑定***。

下面是例子：

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
} 
var obj = { 
    a:2 
}; 
var bar = function() {
    return foo.apply( obj, arguments );
}; 
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

ES5提供了一个内置的硬绑定的方法：`bind()`，用法如下：

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
} 
var obj = {
    a:2 
};
var bar = foo.bind( obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

#### new绑定

使用new来构造函数调用，会发生new绑定。会自动执行下面的操作：

1. 创建一个全新的对象。
2. 这个新对象会被执行[[Prototype]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

参考代码：

```js
function foo(a) {
     this.a = a;
 }
 var bar = new foo(2);
 console.log( bar.a ); // 2
```

`this`的内容大概就这么多，关于这本书对象和原型这部分，书上讲了很多理论知识，并且知识点很多很杂，我的理解也不是很深刻，总结无非是把书上东西搬上来而已，不如换一个方式。慕课网上Bosn老师有一系列课程对对象原型有了很好的解释，不妨去看一下。[链接在这里](http://www.imooc.com/learn/277)

