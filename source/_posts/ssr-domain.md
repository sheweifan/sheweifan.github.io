---
title: SSR 如何做状态透传
date: 2023-02-02
---

Node.js的domain API可以用于处理异步操作中的错误，避免因为一个未处理的错误导致整个进程崩溃。以下是一些常用的domain API：
- domain.create()：创建一个新的域。
- domain.run(fn)：在当前域的上下文中运行给定的函数。
- domain.add(emitter)：将事件发射器添加到当前域中。
- domain.remove(emitter)：从当前域中删除事件发射器。
- domain.bind(callback)：返回一个函数，在调用时将其参数绑定到当前域。
- domain.intercept(callback)：返回一个函数，在调用时将其参数绑定到当前域，并且任何错误都会被捕获并传递给提供的回调函数。
- domain.enter()：将当前域设置为活动域。
- domain.exit()：将当前域恢复为先前的活动域。

使用domain API的示例：
```javascript
  const domain = require('domain');
  const server = require('http').createServer();
  const express = require('express');

  const app = express();
  server.on('request', app);

  // 创建一个新的域
  const d = domain.create();

  // 捕获服务器异常
  d.on('error', err => {
    console.error(err.stack);
  });

  // 将服务器套接字添加到域中
  d.add(server);

  // 在域的上下文中运行服务器
  d.run(() => {
    server.listen(3000);
  });
```
在以上示例中，domain.create()创建了一个新的域。通过将服务器套接字添加到该域中，可以捕获服务器发生的异常，避免因为未处理的异常导致整个进程崩溃。使用domain.run()函数在当前域的上下文中运行代码，保证任何异步操作都在该域中执行，从而捕获所有的错误。