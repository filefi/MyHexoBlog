---
title: 在Tampermonkey中创建并使用WebWorker
date: 2024-04-06 22:35:21
updated: 2024-04-06 22:35:21
tags: [Tampermonkey, UserScript, Front-End, JavaScript, WebWorker]
categories: Tampermonkey
---

```js
// ==UserScript==
// @name         Web Worker Demo
// @namespace    http://tampermonkey.net/
// @version      2024-04-06
// @description  try to take over the world!
// @author       You
// @match        https://www.example.com/
// @grant        GM_setValue
// ==/UserScript==

(function() {
    'use strict';

    // Web Worker 内嵌脚本字符串
    const workerString = `
let count = 1;
setInterval(()=>{
    console.log('worker', count); 
    postMessage({'workerMsg': count}); 
    count++;
}, 3000);
`

    // 由字符串创建Web Worker文件URL
    const workerURL = URL.createObjectURL(new Blob([workerString]));

    // 创建Web Worker
    const worker = new Worker(workerURL);

    // Web Worker无法直接访问GM_*方法，将数据返回给页面主线程
    worker.onmessage = function (e){
        // 由主线程调用GM_*方法
        GM_setValue('workerMsg', JSON.stringify(e.data));
        console.log(e.data);
    }
  
})();
```

![](image-20240406224118761.png)

![](image-20240406224158990.png)

