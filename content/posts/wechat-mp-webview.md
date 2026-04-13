---
title: 小程序内嵌webview
date: 2018-09-04
---

## 一，路由
路由跳转这块，由于公司三端（客户端，移动端，小程序）需要兼容，原先的bridge针对客户端路由做协议跳转，和vue-router配合整理出了一些方法提供调用，所以我们拓展了bridge，加了小程序的路由层。实际业务应用上无需做太多的调整即可做到开关webview来维护路由。

## 二，原生交互
实际使用postMessage效果并不符合期望，postMessage在小程序原生模块很难有即时的触发，这是一个坑点，调试了很久发现不是代码问题，而是没有等到“合适的时机”来触发接收来自webview的信息。由于业务使用比较少交互，目前业务只有需要小程序原生接入支付。
<!--more-->

## 三，调用支付
### 1. 第二点提及postMessage的时效性问题，所以我们并没有使用psotMessage来做桥调用小程序支付。而是使用了一个空白页，类似客户端会openWebview的时候把版本信息，用户验签等信息通过url query的形式传给客户端，小程序借鉴了这个办法，通过url的形式打开一个小程序原生页面来通知小程序端。

### 2.支付调用wx.requestPayment所需的参数（timeStamp，nonceStr，package，signType，paySign）需要接口，而后端去请求微信接口时需要openid。openid获取有两个渠道：
#### 一个是wx.login返回的code去换openid，静默获取。
#### 二是wx.getUserInfo（需 button type="getUserInfo" 做用户拒绝授权的情况下重新弹窗请求授权）返回的encryptedData解密后也有openid。需要用户同意，如果用户不同意，还需要在支付空白页或者其他地方添加按钮来再次获取授权。

因为并无收集用户信息的需求，所以我们选了wx.login来获取openid的方式，之后就是怼接口返回调用requestPayment

## 四，操作webview

我们用了一个专门用来开关webview的小程序原生组件来做容器，通过url上带过去的query作为状态推到状态并体现在webview的src上，传递的src需要做encodeURIComponent。打开和关闭webview都能轻松做到。js-sdk的wx.miniProgram上挂载的navigateTo，navigateBack等方法很方便的去操作。值得一提的是小程序全局对象上的getCurrentPages可以拿到当前所有的页面组件，并且可以直接setData来操作对应页面的状态。