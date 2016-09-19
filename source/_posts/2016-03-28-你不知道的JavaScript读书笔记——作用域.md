---
layout: post
title: 《你不知道的JavaScript》读书笔记——作用域
date: 2016-03-28
tags: ['JavaScript','作用域']
categories: JavaScript	
---

这几天在看《你不知道的JavaScript》这本书，内容简单易懂而且觉得讲的很有意思，趁着还热乎，把刚看完的总结一下吧。这个是第一部分讲的是作用域。

### 1. 初识作用域

作用域是一套规则，用于确定在何处以及如何查找变量。编译器有一个术语，比如我们查找变量a的值，引擎会进行LHS和RHS查询。LHS和RHS是什么呢？通俗的讲就是对赋值操作的左侧和右侧进行查询。

举一个例子，比如对`var a=2`这个赋值操作来讲，左侧和右侧就是`=`等于符号的左侧和右侧（并不一定就是`=`号）。这个`a=2`的赋值操作其实分成两步来进行:
1. `var a`在其作用域中声明新的变量，这个会在执行`var a=2`之前进行。
2. 然后会对`a = 2`进行LHS查询变量a，并对a进行赋值。

所有的查询都会从当前作用域开始查找，如果没有找到这个变量呢？就会逐级向上查找直到全局作用域，如果还是没有找到查找都会停止。

不成功的`LHS`查询（就是没有找到这个变量）就会在全局作用域当中创建一个全局变量（非严格模式下）；不成功的`RHS`（就是没有找到这个变量的值）就会抛出一个异常。

****

### 2. 词法作用域

词法作用域通俗讲，就是我们在编写代码时变量的作用域。我们来考虑下面的代码：

```js
function foo(a){
    var b = a * 2;
    function bar(c) {
        console.log(a , b , c);
    }
    bar( b * 3 );
}

foo( 2 ); // 2, 4 ,12
   
```


这段代码会包含三个逐级嵌套的作用域：
1. 最外层的是全局作用域，其中有一个标识符：foo。
2. 往里是`foo`所创建的作用域，其中有三个标识符：a、bar和b。
3. `bar`所创建的作用域，其中有一个标识符：c。

在程序运行`console.log( a, b, c )`的时候会查找a、b、c这三个变量，会先在当前的作用域开始查找，也就是`bar`的作用域，如果没有找到呢他就会开始往外查找，直到第一个匹配的标识符的时候，他就会停止了。对`a`来说就是在`foo`的作用域找到的，`b`也是；`c`就在`bar`作用域就找到了。

每一层作用域当中呢，都可以存在同名的变量，但是当你找到第一个匹配的变量的时候，外层作用域这个同名的变量就会被遮蔽，这个叫做**遮蔽效应**。
**我们可以通过`window.a`这种方式来访问到被遮蔽的同名全局变量，但是其他被遮蔽的同名变量就访问不到了。

文章中还讲到了`词法欺骗`，但是文章不建议使用它们，我就不叙述了。

******

### 3.函数作用域和块作用域

#### 1.函数中的作用域

函数作用域是说：这个函数内部的变量可以在整个函数范围内使用和复用。比如这段代码：

```js
function foo(a) {
     var b = 2;
     // ...
     function bar() {
         // ...
     }
     // ...
     var c = 3;
}
```

标识符`a`、`b`、`c`和`bar`都属于`foo`这个函数的作用域内部，我们在`foo`函数的外部是无法访问到他们的。

#### 2.隐藏内部实现

利用上面说到的，函数外部是无法访问到内部的变量这个特性可以在设计模块或对象的API的时候使用。

比如：

```js
function doSomething(a) {
     b = a + doSomethingElse( a * 2 );
     console.log( b * 3 );
}
function doSomethingElse(a) {
     return a - 1;
}
var b;
doSomething( 2 ); // 15
```

如果这样来写的话，函数内部的变量都是在全局作用域下的，这个可能会有隐患。可以改变一下，变成这样：

```js
function doSomething(a) {
     function doSomethingElse(a) {
     return a - 1; 
     }
     var b;
     b = a + doSomethingElse( a * 2 );
     console.log( b * 3 );
}
doSomething( 2 ); // 15
```
 
这样变量都在函数的内部了，外部就没有办法访问到了。这样隐藏内部实现还有好处就是可以避免命名冲突，会造成意外的错误。
 
```js
function foo() {
    function bar(a) {
	    i = 3; // 
         console.log( a + i );
     }
     for (var i=0; i<10; i++) {
        bar( i * 2 ); // 会出现错误，无限循环
     } 
}
foo();

```

这就是`i=3`覆盖了for循环中的i，造成了错误。可以改成`var i = 3`或者用不同的变量代替，来修改这个错误。

#### 3.全局命名空间

在加载很多第三方库的时候，为什么不会造成命名冲突，是因为库通常会声明一个对象，而将所有用到的变量，放到这个对象的属性中。

```js
var MyReallyCoolLibrary = {
     awesome: "stuff",
     doSomething: function() {
     // ... 
     },
     doAnotherThing: function() {
     // ...
     } 
};

```

#### 4.函数作用域

我们可以通过隐藏的方式，使外部无法访问到内部的内容：

```js
var a = 2;
function foo() { // <-- 添加这行
     var a = 3;
     console.log( a ); // 3
} // <-- 这行 
foo(); // <-- 这行
console.log( a ); // 2

```

但是这样`foo`这个命名本身就污染了他所在的作用域，而且需要显式的调用，我们可以用立即执行函数表达式来解决这个问题。

***区分函数表达式和函数声明：看`function`关键字的位置，如果`function`是声明的第一个词，那就是函数声明，否则就是函数表达式。***

立即执行函数表达式又叫IIFE（Immediately Invoked Function Expression），它有两种形式：`(function(){...})()`和`(function(){...}())`，两种形式功能是一样。

IIFE还可以传递参数进去：

```js
var a = 2;
 (function IIFE( global ) {
     var a = 3;
     console.log( a ); // 3
     console.log( global.a ); // 2
 })( window );
 console.log( a ); // 2
```
 
IIFE还有另外一种变化：

```js
var a = 2; 
 (function IIFE( def ) {
     def( window );
 })(function def( global ) {
     var a = 3;
     console.log( a ); // 3
     console.log( global.a ); // 2
}); 
```
函数表达式def定义在第二部分，然后当做参数被传递到第一部分中，参数def被调用，把window当做global参数的值传入进去。

#### 5.块作用域

ES3开始，有`try/catch`会有块作用域。

ES6中新加了`let`关键字，他会有隐式的块作用域。

```js
var foo = true;
 if (foo) {
     let bar = foo * 2;
     bar = something( bar );
     console.log( bar );
 }
 console.log( bar ); // ReferenceError
```

但是这样隐式的块作用域不方便阅读，最好是写成显式的块作用域：

```js
var foo = true;
if (foo) {
    { // <-- 显式的块
         let bar = foo * 2;
         bar = something( bar );
         console.log( bar );
    } 
} 
console.log( bar ); // ReferenceError

``` 

***`let`声明不会进行提升，就是在`let`声明的代码被运行之前，声明不会存在。***

```js
 console.log( bar ); // ReferenceError!
 let bar = 2; 

```

除了`let`，ES6还引入了`const`，这个也是用来创建块作用域变量，而且声明的变量值是固定的（常量），修改值的操作会引起错误。

```js
var foo = true;
 if (foo) {
     var a = 2; 
     const b = 3; //  包含在if中的块作用域常量
     a = 3; // 
     b=4;// 错误! 
 } 
 console.log( a ); // 3
 console.log( b ); // ReferenceError!

```

*******

### 4.提升

有这样一段代码：

```js
{
   console.log( a );
   var a = 2 ;
}
```
输出的结果是`undefined`，这就是变量声明的提升。当我们看到`var a = 2`这个变量声明的时候，其实有两步：`var a`和`a = 2;`。第一步在编译阶段进行，第二步赋值步骤会在执行到这句代码的时候在进行。

所以上面的代码其实是这么运行的：

```js
var a;
console.log( a );
a = 2;
```

这个过程就叫做**提升**。

***每个作用域都会进行提升操作，变量的声明都会提升到每个作用域的上方。***

```js
foo(); 
function foo() {
    console.log( a ); // undefined
    var a = 2;
} 
```
上面这段代码来说，全局作用域和foo的作用域都会分别做提升，实际代码执行起来是这样的：

```js
function foo() {
     var a;
     console.log( a ); // undefined
     a = 2; 
}
foo();
```
***函数声明会被提升，函数表达式不会被提升***

```js
foo(); // TypeError
 bar(); // ReferenceError
 var foo = function bar() {
     // ...
}; 
```

如果是一个函数声明的话，变量`foo`其实会被提升，（所以不会报ReferenceError）但是那时没有赋值，`foo`会是`undefined`，会`undefined`进行函数调用时非法的所以报`TypeError`异常。

***函数优先***

```js
foo(); // 1
 var foo;
 function foo() {
     console.log( 1 );
 }
 foo = function() {
     console.log( 2 );
}; 
```

函数声明和变量声明都会被提升，如果一个标识符重复声明为变量和函数，函数会首先被提升。上面的代码其实会被引擎理解成这样：

```js
function foo() {
     console.log( 1 );
} 
foo(); // 1 
 foo = function() {
     console.log( 2 );
}; 
```

虽说`var foo`是在函数声明之前，但是函数声明会提升，并且会忽略掉重读的var声明。

**但是如果是重复的函数声明，后面的函数声明是可以覆盖前面的。**

```js
foo(); // 3 
 function foo() {
     console.log( 1 );
} 
 var foo = function() {
     console.log( 2 );
}; 
 function foo() {
     console.log( 3 );
} 
```





