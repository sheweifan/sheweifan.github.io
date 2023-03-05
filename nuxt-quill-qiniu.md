---
title: 如何在nuxt中使用quill，并关联七牛图床
date: 2018-01-07
---

## 引入
按照[官方ssr demo](https://github.com/surmon-china/vue-quill-editor/tree/master/examples/nuxt-ssr-example) 的做法引入即可。

    import 'quill/dist/quill.snow.css'
    import 'quill/dist/quill.bubble.css'
    import 'quill/dist/quill.core.css'
    
    if (process.browser) {
        const VueQuillEditor = require('vue-quill-editor/dist/ssr')
        Vue.use(VueQuillEditor)
    }
    
<!-- more -->

## 使用
和在vue中使用不同的是，ssr是使用指令的方式引用的。

    div.quill-editor(
        :content="content"
        @change="onEditorChange($event)"
        v-quill:myQuillEditor="editorOption"
    )
    
这样就可以跑起来了，但是有个问题就是通过$ref取到的是dom，并不是组件，所以$refs也不会有quill对象。

## 处理图片
如果你打开控制台，可以看到插入的图片其实是base64格式的，所以我们需要一个上传组件（封装也为了其他地方可以调用）,我使用element-ui来做的。该组件下文称 `uploader` 组件

#### 七牛+el-upload网上有很多文章可以参考，概述一下流程：
- el-upload挂一个 `:data="token"` 状态，发起请求的时候会把这个状态一起发送
- 组件在上传之前会触发`before-upload`事件，在这里需要更新`token`状态，发起请求用前端生成的key去后端换`token`即可，比如nodejs的 `new qiniu.rs.PutPolicy(bucket + ':' + key).token()` ，然后 `this.token={key, token}` 更新状态，然后 `return true` 往下执行
- 七牛会返回一个key，图片的链接就是 `七牛仓库外链域名 + key + 图片处理样式`

> 当然你也可以使用封装好的上传方法，比如 [qiniu-web-uploader](https://github.com/conglai/qiniu-web-uploader)。

#### 使用uploader组件
> 组件封装需要在上传成功之后对外 `$emit('success')`，其他跟本文无关。

上文提到$refs也不会有quill对象，这样就不会有`getSelection`拿到光标位置， `insertEmbed`插入内容了。查[quill文档](https://quilljs.com/docs/modules/toolbar/)发现，api是可以通过 `modules.toolbar.handlers.image` 配置图片点击事件的，但是自带的方法会被覆盖。

思路如下：
- 点击toolbar选择图片的时候，会触发配置的 `modules.toolbar.handlers.image`，把handle的this，挂在组件上`self.quill = this`
- 触发uploader组件，最简单的方式就是 `$refs.uploader.$el.querySelector('.el-upload').click()`
- 选择完图片触发uploader的`@success`，通过 `this.quill.getSelection().index` 拿到光标，然后调用 `this.quill.insertEmbed(光标位置,'image', 图片链接)` 插入图片即可

## 代码

template(pug)

    div
        div.quill-editor(
          :content="content"
          @change="onEditorChange"
          v-quill:myQuillEditor="editorOption"
        )
        uploader(
            ref="uploader"
            @success="uploaderSuccess"
            v-show="false"
        )

script

    export default {
        name: 'editor',
        components: {
          uploader
        },
        data () {
          const self = this
          return {
            content: '',
            editorOption: {
              modules: {
                toolbar: {
                  container,
                  handlers: {
                    image: function() { self.imgHandler(this) }
                  }
                }
              }
            }
          }
        },
        methods: {
          imgHandler (handle) {
            this.quill = handle.quill
            const input = this.$refs.uploader.$el.querySelector('.el-upload')
            input.click()
          },
          uploaderSuccess (file) {
            const index = this.quill.getSelection().index
            const imgUrl = `${imgPrefix}${file.key}?imageView2/0/q/60|imageslim`
            this.quill.insertEmbed(index, 'image', imgUrl)
          }
        }
    }

## 最后
如果你有更好的方法或者对我的思路有疑问，欢迎批评指正。