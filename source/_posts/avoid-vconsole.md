---
title: 如何避免线上显示vconsole
date: 2019-06-07
---


# 问题背景
在现在普遍前端工程化环境下，有时候会使用vconsole来作为移动端调试的工具，但在上线前忘记注释掉vconsole等原因，导致线上显示出vconsole的问题，在看到OA的第n个关于vconsole线上显示的事故报告之后，我觉得应该要有一种手段来自动化的做这样一件事。

# 改进构建
解决方向很显然从构建入手。

<!-- more -->
* webpack config alias配置vconsole别名

      {
        ...
        alias: {
          vconsole: path.resolve(__dirname, 'path/to/vconsole')
        }
      }

* 另起一个vconsole.js，判断如果是打包环境就暴露出去一个空的function，否则才暴露require('vconsole')

      let VConsole = function() {}
      if (!isProd) {
        VConsole = require('vconsole')
      }
      module.exports = VConsole

> 因为全局搜索了vconsole的调用方式，只有 `new VConsole()` 这种调用方式，所以没有对构造函数做进一步拓展。

* 这样修改过构建之后，就可以大胆的调用，条件满足线上，就不会再显示VConsole，也会从打包中被排除。

# 总结
* 每一个需要手动操作的过程，都应该思考是否能被自动化掉。
* ios调试使用 [Mac+Safari](https://www.jianshu.com/p/5b2305929041)，安卓尝试过 [Mobile Debug](https://www.mobiledebug.com/) 体验会比vconsole舒服。
