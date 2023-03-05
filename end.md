---
title: 给你的面试指南
date: 2018-9-26
---

## this
`this`并不难理解，this指向问题，很好理解，谁调用，就指向谁。

    function consoleThis() {
      console.log(this)
    }
    consoleThis() // window || undefined

这段代码，看起来并没有谁在调用`consoleThis`这个函数，但实际上，是`window`在调用，可以理解为，等价于`window.consoleThis()`。
<!-- more -->
非严格模式下，会打印`window`，但是在严格模式下，会打印`undefined`。所以，在定义一个函数的时候，写`this.xxx()`的时候要谨慎，因为严格模式下，`undefined`可没有任何方法给你调用，会报错的。关于严格模式，你可以看看[这篇文章](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)。

    var dog = {
      tag: 'animal',
      eat: function() {
        console.log(this, this.tag)
      }
    }
    var eat = dog.eat
    eat() // window undefined
    dog.eat() // {tag: "animal", eat: ƒ} "animal"

这里的eat方法内的`this`也有不同，
`eat()`是由`window`调用的，所以函数体内的`this`指向`window`。
`dog.eat()`是由`dog`调用的，它指向`dog`，所以可以正确打印出值来。
笔试题有很多这样的坑。

## 闭包
闭包是一个可以用来保存上下文的容器，因为变量被引用所以不会被回收机制回收，所以要注意不要滥用。
具体应用如下:
### 1.保存当前上下文

    for (var i = 0; i < 3; i++) {
      setTimeout(function() {
        console.log(i)
      }, 1)
    }
    
这段代码打印出来肯定会是三个3，因为当定时器执行的时候，for循环已经结束了，当前上下文中的`i`已经变成3，要避免这种情况，可以使用闭包来解决。

    for (var i = 0; i < 3; i++) {
      (function(i){
        setTimeout(function() {
          console.log(i)
        }, 1)
      })(i)
    }

i作为自执行函数里面的变量被保存下来，正确打印了值。在es6下，可以使用`let`代替`var`来解决。关于`let`你可以看一下[这篇文章](http://es6.ruanyifeng.com/#docs/let)。

### 2.权限收敛
在实际开发中，有时候需要把一个变量的权限做限制，比如希望一个变量只能是一个大于0的数字，可以这样做：

    var initCount = function() {
      var _count = 0
      return {
        getCount: function() {
          return _count
        },
        setCount: function(newCount) {
          if (typeof newCount === 'number' && newCount > 0) {
            _count = newCount
          }
        }
      }
    }

    var c = initCount()
    c.getCount() // 0
    c.setCount('string')
    c.getCount() // 0
    c.setCount(2)
    c.getCount() // 2
    console.log(_count) // Uncaught ReferenceError: _count is not defined

上面代码可以看出私有变量不能被直接访问，只能通过`getCount`方法来获取，通过`setCount`来修改，保证了变量的类型和有效值。


## 跨域
跨域只存在于浏览器中，为什么会有跨域，是因为浏览器考虑安全，所以有同源策略。跨域很容易产生，只要协议(http, https)，域名(a.com,b.com)，端口有任何一个不全等，就会产生跨域。

比如，你在你的页面里面嵌入了一个百度`iframe`，你是无法通过dom选择器来找到百度的任何dom元素的。想想，如果你在页面里面嵌入了一个qq登陆的`iframe`，又没有同源策略的话，不就可以轻松获取到访问者输入的账号和密码了吗？这太不安全了。再比如接口请求，如果没有跨域限制，那么你在网上银行登陆了账号，又访问了恶意向网上银行发起请求的网站，那么cookie会泄漏，你的信息就不安全了。

解决跨域的办法有很多种：

### 1.jsonp
你可以简单的理解成，在html里面插入一个script标签，因为scipt标签不受同源策略限制，你插入的这段js可以是`callback({name: 'sheweifan'})`，那你可以在插入这个script标签之前，定义好这个callback方法，插入完成之后，调用了`callback`自然你就可以在事先定义好的函数里面拿到你想要的数据。具体你可以看jQuery的`$.ajax({ataType:'jsonp'})`

### 2.CORS
通过后端设置`Access-Control-Allow-Origin`头，来配置白名单，就不会产生跨域问题了。

### 3.nginx方向代理
了解即可。

还有很多解决跨域的办法，深入了解，不仅可以解决线上环境跨域问题，还可以在开发阶段给你的开发体验带来质的飞跃。


## Get请求和Post请求的区别
- get主要是从服务器获取资源，post主要是像服务器发送数据
- get传输数据通过url请求，利用k=v的形式放在url后面，用?连接，多个用&连接而post是存放在，ajax中的data中的，get传输的过程使用户可见 的，而post是对用户不可见的。
- get传输的数据量小，以为受url的长度限制，但是效率高，post能上传的数据量大
- post较get更安全一些
- get方式传递的中文字符可能会乱码，post支持标准字符集，可以正确传递中文字符

## git使用过程中，如果你在开发着业务，突然另一个分支有一个bug要改，你怎么办
    
    git stash       //将本次修改存到暂存区
    git stash pop  // 取出

另外对于git `sourceTree` 很好用，你可以看看这篇文章[这篇文章](https://mp.weixin.qq.com/s/w93ThkMt-EyFGj9Xp3fw3g)。

## babel
`babel`是一个语法解释器，它只对语法进行编译，对于浏览器不支持的方法，还是需要`babel-polyfill`或者`babel-runtime`。这两者又区别吗？有，`polyfill`是全部引入，而`runtime`是按需引入，你用了什么，就给你兼容什么。

## 内存泄漏
内存泄露，就是在堆上分配了一块内存，由于程序不严谨（如未释放内存），导致指向这块内存的指针无效了，在程序的整个生命周期内都无法被使用。
内存泄露有两种恶性结果，一是泄露的内存比较大，程序会因为内存耗尽而崩溃；二是如果其他应用程序也在这块内存中分配了资源，很可能导致出错甚至崩溃。
如果内存泄露逻辑可以稳定复现，在不断执行这块程序的时候，会持续泄露，系统可用内存可能急剧减少。
来源[@Barret李靖](https://weibo.com/173248656?from=myfollow_all&is_all=1)。