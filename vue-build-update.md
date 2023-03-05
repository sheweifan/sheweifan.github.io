---
title: Vue SPA 项目构建升级
date: 2020-05-05
---

# 背景

项目由 webpack1.x 构建，日常开发过程构建速度较慢，且 babel 存在作用域问题，子目录 babelrc 文件影响项目构建，在折腾了几个小时解决这个问题后，意识到这个项目的构建需要被升级。

# 结果

| -                  | 构建时长 |
| ------------------ | -------- |
| 升级前 webpack 1.x | 91.66s   |
| 升级后 webpack 4.x | 48.97s   |

因为担心风险问题，包内容构建修改前后并无变化，所以包体积基本保持一致。

# 升级过程

### 列出修改点

- 打包去除 vconsole
- postcss px2rem
- htmlWebpackPlugin inject 变量（对模版做写判断，对正式环境切换日志库，百度统计等等）
- 保留打包空格（跟随老配置，否则存在部分样式问题）
- babel 配置升级，移除 es2015 使用 env
- sass resources
- 新增 dll
- 添加 git hooks pretty
  ...

### 替换构建

- 移除老构建目录/build
- `@vue/cli`生成新项目目录

### 遇到的问题

- dev server hot reload 失效

  > vue.config.js 添加 devServer.disableHostCheck: true

- 打包保留空格

  > vue.config.js chainWebpack options.compilerOptions.whitespace = true
