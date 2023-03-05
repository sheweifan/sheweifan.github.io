---
title: rem 适配 系统文字大小
date: 2020-10-18
---
# 问题描述
日常查看反馈日志发现部分安卓用户反馈界面ui错乱，截图显示字体明显大于常规，各个元素都变大了，有些元素撑出了屏幕外。
因此猜测是由于系统/浏览器字体大小被缩放导致。
<!-- <img src="/img/rem-auto-size-feedback.png" style="max-width: 300px" /> -->

# 解决思路
插入一个宽度10rem(屏幕宽度的100%)的div作为依据，如果rem计算正确，这个div的宽度应该和屏幕宽度<!-- more -->一致。如果不一致则根据比例重新计算，延迟再次检查。
为了防止频繁计算rem，如果整个计算过程持续8秒之后，依然无法正确算出正确值，则放弃。

# 代码

        var u = navigator.userAgent
        var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1
        // 只有安卓用户反馈，缩小范围降低风险
        if (isAndroid) {
          // 全屏幕宽度10rem
          var remFull = 10
          var eDivWidth = 0
          var eDiv = document.createElement('div')
          eDiv.style.width = remFull + 'rem'
          eDiv.style.height = '1px'
          eDiv.style.position = 'fixed'
          eDiv.style.boxSizing = 'border-box'
          document.body.appendChild(eDiv)
          var startTS = +new Date()
          var timer = setInterval(() => {
            // 创建一个10rem宽度的div，这个div的宽度应该与屏幕宽度一致。不一致就是 字体被调大了
            eDivWidth = eDiv.clientWidth
            var clientWidth = docEl.clientWidth
            // 为了解释把 || 改成了两个if
            if (+new Date() - startTS >= 8 * 1000) {
              clearInterval(timer)
              document.body.removeChild(eDiv)
              console.log(`rem 重新计算 任务超时`, rem)
            } else if (clientWidth === eDivWidth) {
              clearInterval(timer)
              document.body.removeChild(eDiv)
              console.log(`rem 没问题了`, rem)
            } else {
              console.log(`rem 有问题，重新设置，错误的rem是`, rem)
              rem = rem * (clientWidth / eDivWidth)
              docEl.style.fontSize = rem + 'px'
              flexible.rem = win.rem = rem
            }
          }, 100)
        }   

# 思考
虽然后续反馈问题得到解决，但是这样这并不友好，毕竟用户改了系统字号设置，就是想看更大的字体。