---
title: 微信小程序如何捕获返回
date: 2022-12-07
---

# 需求背景
在开发过程中，总会有需求是在用户返回前做一些事情，而目前小程序提供的并没有直接支持，需要点奇淫巧技才可以。

# 解决方案
利用`page-component`的`bind:beforeleave`来捕获到页面返回。整体思路：
<!--more-->
- 在页面插入一个`page-container`, `z-index`设置成-1，把页面其他元素设置定位，覆盖在该组件上，将`page-container`隐藏起来。
- 添加`bind:beforeleave`事件监听，再用户触发返回时可以捕获到。
- 用户触发返回后，`page-container`组件会被销毁，需要重新插入。可以利用`setDate` 和 `wx:if` 来实现。
这有就实现了对返回操作的监听

# 代码实现
```html
<page-container
  wx:if="{{isHackGobackTrackPageComponent}}"
  show="{{true}}"
  bind:beforeleave="beforeleave"
  z-index="{{-1}}"
>
</page-container>
```
```js
{
  data: {
    ...
    isHackGobackTrackPageComponent: true
    ...
  },
  ...
  beforeleave() {
    this.setData(
      {
        isHackGobackTrackPageComponent: false
      },
      () => {
        setTimeout(() => {
          this.setData({
            isHackGobackTrackPageComponent: true
          })
        }, 100)
      }
    )
    wx.showModal({
      title: '提示',
      content: '确认要退出吗？',
      success: res => {
        if (res.confirm) {
          console.log('用户点击确认')
          // TODO
        } else if (res.cancel) {
          console.log('用户点击取消')
        }
      }
    })

  }
  ...
}
```

# 相关API
## disableSwipeBack
禁止滑动返回，只能通过点击，这个时候就可以捕获到退出页面，但功能已下线 
## wx.enableAlertBeforeUnload
`enableAlertBeforeUnload` 是小程序提供的开启小程序页面返回询问对话框。功能局限性太高，且安卓手势返回并不支持。

# 相关阅读
- [component · page-container](https://developers.weixin.qq.com/miniprogram/dev/component/page-container.html)
- [API · wx.enableAlertBeforeUnload](https://developers.weixin.qq.com/miniprogram/dev/api/ui/interaction/wx.enableAlertBeforeUnload.html)
- [“右滑手势返回”能力调整](https://developers.weixin.qq.com/community/develop/doc/000868190489286620a8b27f156c01?highLine=disableSwipeBack)
- [小程序监听返回、阻止页面返回、弹框后禁止返回](https://blog.csdn.net/qdm13209211861/article/details/126973408)