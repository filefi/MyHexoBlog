---
title: 批量将某li某li特定关注组中的UP修改为悄悄关注
date: 2022-06-01 23:21:23
updated: 2022-06-01 23:21:23
tags: [JavaScript, reverse, PWN]
categories: JavaScript
---

如题，调用batchModify函数即可批量将某li某li特定关注组中的UP修改为悄悄关注。

<!-- more -->

```js
/**
*将bilibili某个关注组中的所有up批量改为悄悄关注
*@param mid {number} 自己的id
*@param tagid {number} 要被批量修改的关注组的组id
*/
async function batchModify(mid, tagid, maxPageNum=100) {
    let map = new Map();
    document.cookie.split('; ').forEach(item=>{return map.set(item.split('=')[0], item.split('=')[1])});
    const csrf = map.get('bili_jct');
    for (let pn = 1; pn <= maxPageNum; pn++) {
        const resp = await fetch(`https://api.bilibili.com/x/relation/tag?mid=${mid}&tagid=${tagid}&pn=${pn}&ps=20&json=json&callback=__jp16`, {
        "headers": {
          "accept": "*/*",
          "accept-language": "zh-CN,zh;q=0.9",
          "cache-control": "no-cache",
          "pragma": "no-cache",
          "sec-ch-ua": "\" Not A;Brand\";v=\"99\", \"Chromium\";v=\"101\", \"Google Chrome\";v=\"101\"",
          "sec-ch-ua-mobile": "?0",
          "sec-ch-ua-platform": "\"Windows\"",
          "sec-fetch-dest": "script",
          "sec-fetch-mode": "cors",
          "sec-fetch-site": "same-site"
        },
        "referrer": `https://space.bilibili.com/${mid}/fans/follow?tagid=${tagid}`,
        "referrerPolicy": "no-referrer-when-downgrade",
        "body": null,
        "method": "GET",
        "mode": "cors",
        "credentials": "include"
        });
        const json = await resp.json();
        if (json.data.length === 0){
            return;
        }
        for (let item of json.data) {
            console.log(item.mid);
            await modify(item.mid, csrf);
        }
    } 
}

/**
* 将用户id为fid的up注改为悄悄关注
* @param fid {number} 要修改为悄悄关注的up的用户id
*/
function modify(fid, csrf) {
    return fetch("https://api.bilibili.com/x/relation/modify", {
      "headers": {
        "accept": "application/json, text/plain, */*",
        "accept-language": "zh-CN,zh;q=0.9",
        "cache-control": "no-cache",
        "content-type": "application/x-www-form-urlencoded",
        "pragma": "no-cache",
        "sec-ch-ua": "\" Not A;Brand\";v=\"99\", \"Chromium\";v=\"101\", \"Google Chrome\";v=\"101\"",
        "sec-ch-ua-mobile": "?0",
        "sec-ch-ua-platform": "\"Windows\"",
        "sec-fetch-dest": "empty",
        "sec-fetch-mode": "cors",
        "sec-fetch-site": "same-site"
      },
      "referrer": "https://space.bilibili.com/14629610/?spm_id_from=333.999.0.0",
      "referrerPolicy": "no-referrer-when-downgrade",
      "body": `fid=${fid}&act=3&re_src=11&spmid=333.999.0.0&extend_content=%7B%22entity%22%3A%22user%22%2C%22entity_id%22%3A14629610%2C%22fp%22%3A%220%5Cu00011920%2C%2C1080%5Cu0001Win32%5Cu000116%5Cu00018%5Cu000124%5Cu00011%5Cu0001zh-CN%5Cu00011%5Cu00010%2C%2C0%2C%2C0%5Cu0001Mozilla%2F5.0%20%28Windows%20NT%2010.0%3B%20Win64%3B%20x64%29%20AppleWebKit%2F537.36%20%28KHTML%2C%20like%20Gecko%29%20Chrome%2F101.0.4951.67%20Safari%2F537.36%22%7D&jsonp=jsonp&csrf=${csrf}`,
      "method": "POST",
      "mode": "cors",
      "credentials": "include"
    })
}
```