---
title: azure openai如何做流式输出（打字效果） 
date: 2023-07-02
---

# 区别
openai和azure openai最大的区别就是不需要科学上网，且据azure openai代理商介绍，微软在国内已经完成了合规化备案，只要不上传用户数据，无需担心法律问题。

# 如何从openai 切换到 azure openai

## 调用方式不同（基于npm仓 openai@3.3.0）
azure版本的api不再需要在调用时传入OPENAI_API_KEY，而是在请求中带在header里
basePath也需要根据新建的EndPoint（终结点）以及Studio中新建的model（模型）来决定，大致是${EndPoint}openai/deployments/${modelName}

以axios为例
```js
const { Configuration, OpenAIApi } = require('openai')
const axios = require('axios')

const http = axios.create({
  headers: {
    'api-key': process.env.OPENAI_API_KEY
  },
  params: {
    'api-version': '2023-05-15'
  }
})
const OPENAI_BASE_PATH = `${EndPoint}openai/deployments/${modelName}`
const openai = new OpenAIApi(undefined, OPENAI_BASE_PATH, http)

module.exports = openai
```

## stream流区别
由于azure的接口目前虽然支持stream的参数，但是返回的时候不想openai那样，而是卡顿一会，一次返回一大段，而且输出格式也很奇葩，流是中断的

![Alt text](/img/azure-openai-output.png)

如图，返回的数据格式化后是数组，且JSON.parse报错，原因是数据最后一段数据可能需要下一个数组的第一段数据合并后才算完整的JSON string...

解决办法，拼接流
```js
const handleStream = (completion, stream) => {
  let cache = Buffer.from('')
  let preSend = ''
  completion.data.on('data', data => {
    cache = Buffer.concat([cache, data])

    const lines = cache
      ?.toString()
      ?.split('\n')
      .filter(line => line.trim() !== '')
    let result = ''
    let isEnd = false

    for (const line of lines) {
      const message = line
        .toString()
        .trim()
        .replace(/^data: /, '')

      // 流结束
      if (message === '[DONE]') {
        console.log('流结束')
        // stream.end()
        isEnd = true
        // return
      }

      // 解析数据
      try {
        const parsed = JSON.parse(message)
        // 打印有效内容
        // console.log('=>', parsed.choices[0].delta)
        if (parsed.choices[0].delta && parsed.choices[0].delta.content) {
          result += parsed.choices[0].delta.content
        }
      } catch (e) {
        console.log(`e`, e)
      }
    }
    result && stream.write(result.replace(preSend, ''))
    isEnd && stream.end()

    preSend = result
  })
}

handleStream(completion, stream)
```
