---
title: 关于gulp你可能想知道这些
date: 2018-01-01
---
## 关于gulp
动手之前只要了解两个概念和五个api

- 概念
    - 总是返回stream
    - 组合任务

<!-- more -->

- api
    - src 读文件
    - dest 写文件
    - watch 监听文件
    - pipe 传递stream
    - task 可组合任务 

## 插件
- require-dir 引入任务
- shelljs 命令行工具
- yargs 拿到命令行参数
- gulp-if 条件判断
- browser-sync 静态服务器
- gulp-plumber 防止异常退出
- gulp-debug 输出信息
- gulp-rename 重命名
- gulp-cached + gulp-remember 增量编译

## 目标
### pug 处理模板
> 用过pug之后爱不释手，免去标签闭合，再也不用为了一个少些一个/找半天了。
1、享受公用模板的福利，extends模板进页面之后block对各个块插入内容即可
2、可以在页面定义变量，each，if等等避免冗长重复的代码
    
### stylus 处理样式
> 和pug一样，免去{}，选择器嵌套，可编辑函数
    
### es6/7 处理js
> es6/7福利
模块化避免代码太长，可以把方法抽出来公共，需要的时候引用，既不需要在页面引用很多js，而且代码阅读性更好
    
## 可能遇到的问题

- 项目目录如何规划
> 一个dev目录，一个发布目录。我的项目都是监听/app目录编译到/dist目录

- 我只想更新当前更改的文件，不想每次保存都更新
> 样式和模板可以用cached+remember可以做到，js试了n多插件，不论是changed，watch还是会全量跑（日志输出n条），cached+remember只会更新本文件，修改文件依赖的时候并不会更新。刚好我用webpack-stream使用他的watch机制，顺便用他的source-map

- 独立打包js后拿不到变量
> import 依赖 挂在window上，配置webpack-stream的externals即可，类似引用cdn的做法

- es6/7不支持报错
> 如果是bable的方法未定义，babel.plugins配置上transform-runtime即可

- 更改即更新
> browser-sync.create().reload方法，在dest结束后调用即可，如果是gulp-watch需要在回调里面调用。可能需要你的serve已经开启，如果出错，可以尝试在组合任务的前后顺序入手解决

- 编译太慢
> 如果有这方面的担心，可以少做一些在dev阶段不用做的事，比如imagemin，在dev阶段没必要做，直接把图片dest过去就可以，build的时候再做。比如小图片转base64，dev阶段没必要做。还有上面说的只编译修改文件，都可以加快编译速度。大项目后期编译时间最长的是js，有很大的空间可以优化。总之gulpif先用起来

- 区分编译阶段
> 用yargs来做，如果很多参数的话可以加上npm script。yargs配置完export出去让别的文件require使用。可以配置默认项非常方便

- 陷入修改，报错的循环
> plumber可以pass掉错误，只是一个折中方法，免去异常退出手动重启的麻烦

- 依赖文件也被打包了
> 可以排除目录的方式来做到，比如src`'js/**/*.js'`，可以再加条规则，`!'js/**/lib/*.js'`。我的做法是，需要的打包的文件加上.entry，比如index.entry.js，通过rename把.entry去掉然后再dist。因为我可能一个页面复杂了，会把模块拆得很细。比如list.pug，side.pug，side.styl，listRender.js等等，这样做还有一个好处，就是你其他页面需要引用的时候，你可以把这个模块放到公共文件夹里面，然后修改引用来做到更好维护

- cached+remember无效
> 只要在src后pipe cached，在dest之前pipe remember即可。另外建议在watch onChange的时候，判断如果是删除文件的操作，把该文件的cached，remember踢出

## 我的代码
github [gulp-start](https://github.com/sheweifan/gulp-start)
> 调试阶段，欢迎试用反馈