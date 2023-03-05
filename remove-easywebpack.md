---
title: SSR移除easywebpack
date: 2019-11-09
---

# 背景
改造的是一个vue ssr项目，基于Egg.js+easywebpack搭建而成，已稳定运行一年多。但在这期间发现easywebpack 搭建起来的构建很难升级，甚至很多官方文档标注的功能都实现不了。且这是一个个人维护的项目，在阿里内部已被废弃。有问题也只能自己解决，没有任何官方反馈。

# easywebpack 存在的问题
  * webpack.config.js 里无法拿到打包对象（target）。
  * 分包困难，在github issues看到有人提出一样的问题，但是作者提供的解决方案无法实现，不管怎么配置，甚至重写loader，都无法做到样式抽离。
  * easy-team 配套的插件搭配出来只有一份manifest，在服务端分包直出时无法内联对应的css,js，首屏会无样式闪烁。
  * 打包缓存无法生效，打包时如果没清理缓存（easy clean all），打包报错。（clean all 是作者在issues给的建议）
  * dll在dev阶段经常抽风，经常dll报未定义，要重启服务
  * 打包和渲染服务耦合了，剔除其中一个比较困难。

# 替换方案：vue-cli
  * 有专门的cli团队维护，出问题基本上都能issues搜到官方回应，提issues也很快会收到答复
  * 配置灵活
  * 插件系统，vue add用起来非常方便

# 保留 Egg.js
Egg.js 是基于Koa，Koa是一个包括了后续所有中间件的装饰器模型，但是用在实际项目中还是略显单薄。