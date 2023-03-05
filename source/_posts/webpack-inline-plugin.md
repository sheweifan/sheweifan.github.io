---
title: webpack 文件内联
date: 2020-05-05
---

# 背景

日常开发过程中很多 html 文件只有短短几行 html。而一些细碎的 css,js 引用造成首屏请求数过多。
希望有一个机制来把这些细碎文件内联进 html。
搜索了 webpack 拓展库并没有插件很好的匹配当前的需求，所以需要自己写 plugin。

# 需求

自动的对 webpack 打包出来的文件做内联，通过传入最大内联尺寸来限制。
另外可以通过传入其他非打包文件，这些文件需要跳过最大内联尺寸限制。
<!-- more -->

所以初期传入的 config 是这样的：

| 参数 | 说明 | 类型 |
| ------- | ------------------------------------ | -------- |
| maxSize | 内联最大文件尺寸 | number |
| js | 除了 webpack 打包文件之外的 js 文件 | [string] |
| css | 除了 webpack 打包文件之外的 css 文件 | [string] |

# 开发

## 如何写一个 webpack plugin?

根据官方文档，一个自定义的 plugin 需要包含：

- 一个 javascript 命名函数
- 插件函数的 prototype 上要有一个 apply 方法
- 指定一个绑定到 webpack 自身的事件钩子
- 注册一个回调函数来处理 webpack 实例中的指定数据
- 处理完成后调用 webpack 提供的回调

## 思路

打包文件的文件部分：在 webpack 开始构建后，获取 htmlWebpackPlugin 周期钩子，监听 alterAssetTagGroups，获取静态资源，判断文件尺寸，用文件 source 替换 url
引入的 js css 部分：读取文件后直接插入

## 基本代码

    class InlineChunkHtmlPlugin {
      apply(compiler) {
        compiler.hooks.compilation.tap("InlineChunkHtmlPlugin", compilation => {
          // 开始构建

          // 处理htmlwebpackplugin即将内联的资源
          const hooks = this.htmlWebpackPlugin.getHooks(compilation);
          hooks.alterAssetTagGroups.tap("InlineChunkHtmlPlugin", (assets) => {
            // 对以下两个资源进行处理后替换即可
            assets.headTags // 将会被插入到头部的资源
            assets.bodyTags // 将会被插入到头部的资源

            // 对传入对js css静态资源处理
            for (let i = 0; i < js.length; i++) {
              assets.bodyTags.unshift({
                tagName: "script",
                innerHTML: extractSource(js[i]), // 读文件方法 fs.readFileSync().toString()
                closeTag: true,
              });
            }
            // css同理
          }
        })
      }
    }

# 代码仓库

[https://github.com/sheweifan/webpack-inline-source-plugin](https://github.com/sheweifan/webpack-inline-source-plugin)
