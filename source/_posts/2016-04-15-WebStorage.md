---
layout: post
title: WebStorage
date: 2016-04-15
tags: ['WebStorage','页面储存']
categories: 工作总结	
---

接着上周那个获取地理位置的功能继续说，上周JSSDK注册是成功了，但是又发现了另一个bug，如果一个未登陆过的用户从分享链接登录会报错误，state参数过长，登录失败。

这个bug出现的原因是在后台，因为后台代替前端去调微信授权的接口，取到code后再去种session到前端的这个页面，后台会携带一些参数在state这个字段中，但微信state字段最大支持128字节，超出这个最大限制后就会报错了。

*我不太清楚在state字段中带参数的原因，但是听他们讨论大概是因为需要跨域去传递一些东西，所以要在state字段中传递参数。

后来就改变了注册的方式，原来是后台去代替前端去调微信授权的接口，现在改为：
1. 首先如果我检测到没有session的话，我会去调微信授权的接口，回调域写我本身，这样通过授权后跳回到页面url后面会带一个code的参数。
2. 拿着url后code这个参数，再去请求后台的接口，后台接收两个参数：一个code和回调的url。
3. 把code和回调的url（还是我页面本身）传到后台，后台就会在应答头中把session给我带回到页面中。

****

### Storage

由前端来处理的时候，可能会用到一样东西就是本地储存，为什么会用到？因为我们在调两个接口的时候回调的url都是同一个（即最初页面的url，而不是携带code参数的url），而如果页面重新加载后js会重新执行，并不能缓存一些参数，如果你使用`location.href`来取url的话，不能保证这个url是一致的，当然也可以通过js处理来把后面不需要的参数去掉，但是那么做显然会繁琐一些。

如果用到本地存储的话，我们可以把要用到的url存储在本地，当页面再次加载是从Storage中读取那个想要的url就可以了。

我是这么做的：

```js
  var code = util.getQuery(location.href).code;

  if(util.Cookie.get('sessionid') == ''){
    if(!code){
      localStorage.redirect_uri = encodeURIComponent(location.href);
      location.href = 'https://o2.qfpay.com/trade/wechat/v1/get_weixin_code?appid=wxeb6e671f5571abce&redirect_uri='+localStorage.redirect_uri+'&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect' ;
    }else{
      location.href = config.host + 'wx_callback?redirect_url=' + localStorage.redirect_uri + '&code=' + code;
    }
  }
```

首先我会判断url中有没有code，如果没有那说明这个url是没有经过微信授权的那个url，我就会把这个url写到localStorage中。如果有code，我就把code连同要回调的url传到后台接口去。

主要是介绍一下localStorage和sessionStorage：（参考犀牛书）

localStorage和sessionStorage都是用来存储数据的属性，两者的区别在于存储的有效期和作用域的不同：数据可以存储多长时间以及谁拥有数据的访问权。

#### 有效期

localStorage存储的数据是永久性的，除非刻意的去删除存储的数据（浏览器的清除功能等），否则数据一直保留永不过期。

sessionStorage存储的数据是暂时性的，如果当前窗口或者标签页被永久关闭了，通过当前页sessionStorage存储的数据就会被删除掉。（现在浏览器有恢复最近关闭的标签页的功能，所以sessionStorage得有效期可能会长一些）。

#### 作用域

localStorage的作用域是同文档源，文档源由协议、主机名和端口确定的，三个当中任意一个不同都不属于同文档源。

```js
http://www.example.com  //协议：http；主机名：www.example
```

同源的文档间可以共享localStorage数据，并且可以互相读写，反之则不可以。但是注意localStorage的作用域也受浏览器的限制，不同浏览器只能访问本浏览器localStorage存储的数据。

sessionStorage除了遵循上面的同源策略之外，作用域还被限制在窗口的标签页中。两个不同的标签页各自拥有各自的sessionStorage数据不能共享，就算两个标签页渲染的是同一个页面运行同一个脚本也不可以。

如果一个标签页包含两个`<iframe>`的话，如果它们所包含的文档是同源的，两者的sessionStorage是可以共享的。

*****

localStorage和sessionStorage目前只支持存储字符串类型的数据。存储其他类的话需要自己进行编码和解码。

```js
//读写数字
localStorage.x = 10;
var x = parseInt(localStorage.x);

//读写日期
localStorage.lastRead = (new Date()).toUTCString();
var lastRead = new Date(Date.parse(localStorage.lastRead));

//读写JSON
localStorage.data = JSON.stringify(data);
var data = JSON.parse(localStorage.data);
```

localStorage和sessionStorage除了可以通过设置属性和查询属性来读写之外，也有正式的API。

```js
localStorage.setItem("key","value");//以“key”为名称存储一个值“value”
localStorage.getItem("key");//获取名称为“key”的值

//枚举localStorage的方法：
for(var i=0;i<localStorage.length;i++){
     var name = localStorage.key(i)​;
     var value = localStorage.getItem(name);​
}

//删除localStorage中存储信息的方法：
localStorage.removeItem("key");//删除名称为“key”的信息。
localStorage.clear();​//清空localStorage中所有信息
```
