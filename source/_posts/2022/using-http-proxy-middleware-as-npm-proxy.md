---
title: 使用 http-proxy-middleware 创建 npm 代理
date: 2022-06-09 23:34:01
updated: 2022-06-09 23:34:01
tags: [Proxy, Node.js, NPM]
categories: Node.js
---

在上一篇中，我们使用 [goproxy](https://github.com/snail007/goproxy/releases) 创建 HTTP 代理，供内网中服务器使用 npm 下载并安装第三方外部依赖。其实，借助 `http-proxy-middleware` 包，我们也可以使用 Node.js 来创建 HTTP 代理。

<!-- more -->

**安装依赖**
```bash
npm i express http-proxy-middleware
```

**代码实现**
```js
// proxy.js

// include dependencies
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const app = express();

// proxy middleware options
/** @type {import('http-proxy-middleware/dist/types').Options} */
const options = {
  target: 'https://registry.npmmirror.com', // target host with the same base path
//   changeOrigin: true, // needed for virtual hosted sites
  logger: console,
  secure: false, // don't verify the SSL certs
};

// create the proxy
const proxy = createProxyMiddleware(options);

// mount `exampleProxy` in web server
app.use(proxy);
app.listen(33080);
```

**修改 npm registry**

由于我们的实现没有使用HTTPS，需要修改 npm registry 为`http://`。
```
npm config set registry http://registry.npmmirror.com
```

**测试**

在开发用的电脑上运行我们实现的代码：
```bash
node proxy.js
```

在服务器上使用 npm 安装外部依赖包：
```bash
$ npm i npm -g

changed 14 packages in 3s

11 packages are looking for funding
  run `npm fund` for details
```

```bash
$ npm i nrm -g
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142

changed 73 packages in 6s

11 packages are looking for funding
  run `npm fund` for details
```

可以看到，内网服务器同样可以通过我们实现的 HTTP 代理使用 npm 安装第三方外部依赖了。