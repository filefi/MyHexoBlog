---
title: 使用 goproxy 为处于内网环境的开发服务器提供npm代理
date: 2022-06-09 20:40:55
updated: 2022-06-09 20:40:55
tags: [goproxy, Proxy, 代理]
categories: 
---

通常，企业内部使用的 Web 应用，其服务器通常无法直接与外网通信，导致没法直接在服务器上使用 npm 或 yarn 等工具安装第三方外部依赖。而开发使用的电脑通常是可以访问互联网的，那么，就可以在开发电脑上使用 goproxy 创建一个 http 代理，通过这个 http 代理临时为服务器提供外网访问，以便可以使用 npm 或 yarn 安装第三方外部依赖。

**环境**
- 开发计算机：
  - Windows 10
  - 可以访问服务器，也可访问互联网
  - IP: 192.168.200.1
- 服务器：
  - CentOS 8 
  - 仅可访问开发主机，无法直接访问互联网
  - IP: 192.168.200.129

<!-- more -->

**具体操作**

第1步：在开发计算机上使用 [goproxy ](https://github.com/snail007/goproxy/releases) 创建 http 代理：
```cmd
.\proxy.exe http -t tcp -p "192.168.200.1:33080" --max-conns-rate 0 --forever
```

第2步：在服务器上启用全局http代理。注意，以下为临时启用，直接设置`http_proxy`和`https_proxy`环境变量。如果需要重启后依然生效，则需要将其写入配置文件。
> 注意：这里 `http_proxy` 和 `https_proxy` 都是指向 `http://` 而不是 `https://` 。
```bash
export http_proxy=http://192.168.200.1:33080
export https_proxy=http://192.168.200.1:33080
```



第3步：配置 NPM 代理，将 `proxy` 和 `https-proxy` 同时指向 `http://192.168.200.1:33080`。
> 注意：这里 `proxy` 和 `https-proxy` 都是指向 `http://` 而不是 `https://` 。
```bash
npm config set proxy http://192.168.200.1:33080
npm config set https-proxy http://192.168.200.1:33080
```

第4步：修改 NPM 源（非必需）。
```bash
npm config set registry https://registry.npmmirror.com
```

**测试**

使用 npm 安装 `nrm` ，测试 http 代理是否可用。
```bash
$ npm i nrm -g
npm WARN config global `--global`, `--local` are deprecated. Use `--location=global` instead.
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142

changed 73 packages, and audited 316 packages in 10s

13 packages are looking for funding
  run `npm fund` for details

4 vulnerabilities (1 moderate, 1 high, 2 critical)

To address all issues, run:
  npm audit fix

Run `npm audit` for details.
```
可以看到处于内网的服务器已经成功安装了 `nrm` 包。同时，开发计算机的控制台也输出了来自服务器的相关 log ： 

```cmd
 E:\proxy-windows-amd64>.\proxy.exe http -t tcp -p "192.168.200.1:33080" --max-conns-rate 0 --forever
2022/06/09 20:36:25.234653 INFO worker E:\proxy-windows-amd64\proxy.exe [PID] 54012 running...
2022/06/09 20:36:25.343617 INFO tcp http(s) proxy on 192.168.200.1:33080
2022/06/09 20:36:28.837921 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:28.837921 INFO use parent : false, registry.npmmirror.com:443
2022/06/09 20:36:28.854774 INFO conn 192.168.200.129:55558 - 39.130.171.71:443 connected [registry.npmmirror.com:443]
2022/06/09 20:36:28.999728 INFO conn 192.168.200.129:55558 - 39.130.171.71:443 released [registry.npmmirror.com:443]
2022/06/09 20:36:29.091544 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:29.091544 INFO use parent : false, registry.npmmirror.com:443
2022/06/09 20:36:29.091544 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:29.091544 INFO use parent : false, registry.npmmirror.com:443
2022/06/09 20:36:29.096564 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:29.096564 INFO use parent : false, registry.npmmirror.com:443
2022/06/09 20:36:29.097565 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:29.097565 INFO use parent : false, registry.npmmirror.com:443
2022/06/09 20:36:29.098247 INFO conn 192.168.200.129:55560 - 39.130.171.71:443 connected [registry.npmmirror.com:443]
2022/06/09 20:36:29.098753 INFO conn 192.168.200.129:55562 - 39.130.171.71:443 connected [registry.npmmirror.com:443]
2022/06/09 20:36:29.102769 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:29.102769 INFO use parent : false, registry.npmmirror.com:443
2022/06/09 20:36:29.102903 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:29.102903 INFO CONNECT:registry.npmmirror.com:443
2022/06/09 20:36:29.102903 INFO use parent : false, registry.npmmirror.com:443
2022/06/09 20:36:29.102903 INFO use parent : false, registry.npmmirror.com:443
```