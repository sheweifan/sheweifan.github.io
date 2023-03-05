---
title: webpack1 升级到 webpack5
date: 2022-02-29
---

# 项目背景
一个五年持续迭代但是构建没有升级过的项目，页面多达265个。用vue-cli1开出来的项目，基于webpack1，新增插件基本无法兼容。构建速度持续递增。

# 优化结果
时间统计于统一配置下的容器内打包

|  |首次|第二次无修改|第二次路由页面修改|
|----|----|----|
|webpack1|6 分 7 秒|4 分 0 秒|3 分 56 秒|
|webpack5|1 分 57 秒|13 秒| 57 秒|

<!-- more -->

# 升级过程
待完善，先记录遇到的问题

# 问题
## hot reload 无效

```
// webpack.config.js
{
  ...
  target:web,
}
```

---

## 设置 target:web 后， dev server 启动后浏览器报错 arguments is not defined

检查相关代码是否在箭头函数里使用了 arguments，改过来即可

---

## export 'default' (exported as 'default') was not found in '-!../../../../node_modules/mini-css-extract-plugin/dist/loader.js!../..

```
// webpack.config.js
{
  loader: MiniCssExtractPlugin.loader,
  options: {
    esModule: false,
  },
}
```
参考：https://github.com/vuejs/vue-loader/issues/1742#issuecomment-715882675

---

## exports is not defined
````
// commonjs esmodule 混打？
// babel.config.js
{
  ...
  plugins: [
    ...
    '@babel/plugin-transform-modules-commonjs',
  ]
}
// 尽管加了插件，仍不支持一个文件混用两种导出模块的方式
````
参考：https://www.npmjs.com/package/@babel/plugin-transform-modules-commonjs

---

## Module not found: Error: Can't resolve 'timers' in xxxxx

```
// yarn add timers-browserify
// webpack.config.js
resolve: {
  ...
  fallback: {
    timers: require.resolve('timers-browserify'),
    fs: false
  }
}
```
参考：https://www.npmjs.com/package/timers-browserify

--- 
## 设置port 80之后，macOS下仍然跑1024端口

mac下需要加sudo才可以跑1024以下的端口，加上sudo命令即可

---
## host/charles 代理之后 dev hot reload 失效

```
// webpack dev server
{
  ...
  devServer: {
    ...
    disableHostCheck: true,
 }
}

```
---
## esmodule/commonjs 混打， vant 重复打包

```
// babel.config.js

// 调整插件位置
{
  plugins: [
    '@babel/plugin-transform-modules-commonjs',
    '@babel/plugin-transform-runtime',
    [
      'import',
      {
        libraryName: 'vant',
        style: true
      },
      'vant'
    ]
  ]
}

// 修改为
{
  ...
  plugins: [
    [
      'import',
      {
        libraryName: 'vant',
        style: true
      },
      'vant'
    ],
    '@babel/plugin-transform-modules-commonjs',
    '@babel/plugin-transform-runtime',
  ]
}

```

--- 
## 热更新失败

删除devServer.watchfiles
---