---
layout: post
color: 4a148c
title: 一套将域名解析到局域网的免费方案
author: 扑兔
description: 付费的办法千千万，免费的办法有点难。
---

## 首先是避免走弯路的框架

- 如果你有公网固定 ip，直接用就行了
- 如果你有不固定的公网 ip，可以在路由器上部署 DDNS
- 如果你仅有内网 ip，那么要考虑 ngrok 之类的方案
  1. ngrok 有付费的服务，可以绑定固定域名
  2. 如果你不想选择任何付费的服务，那么本文的路线可以借鉴

## 前言

小项目部署在树莓派上测试，能指个二级域名会方便许多。近日摸索出一条道路可以免费解决，自上而下的流程是：  

- 域名交给 Cloudflare 绑定到 [Wokers](https://workers.cloudflare.com/)
- wokers 会返回一个页面，自动跳转到全局变量设定好的链接，也就是 ngrok 免费套餐给出的随机链接
- 全局变量怎么自动设定呢？
  1. 配置树莓派定时启动一个 sh 脚本，这个脚本会启动 ngrok 并获取给出的随机链接
  2. 上面这个 sh 脚本会判断 ngrok 链接是否变化了，如果变了的话，通过下面的办法更新
  3. Cloudflare 给出的 api 可以提交 Workers 的代码和全局变量

经过这么一折腾，就能够全自动了。开心 ;-)  
解释一下：Cloudflare Workers 是边缘计算应用，也算是无服务器的应用平台。现在许多云平台都在搞，但是有不同的级别，有些需要你自己构建 dockerfile，有些仅需要写 flask 的部分，而这个 Workers 只需要写处理流量的函数，类似的产品像 GCP 的 Cloud Function。Workers 需要 用 js 写，不熟悉没关系，照猫画虎就够了。  

## 具体遇到的问题

树莓派上需要写一个 always_online.sh 脚本，由你决定他运行的频率(crontab)，里面的内容包括：  
后台启动 ngrok  
`screen -dmS ngrok bash -c './ngrok http 80';`

尝试找出 https 链接，如果十次没找到，就终结 screen，下次再说  

``` shell
for i in {1..10}; do
    url=`curl -s localhost:4040/api/tunnels | jq -r .tunnels[].public_url | grep https`
    if [ "$url" != "" ]; then
        echo '{"body_part":"script","bindings":[{"type":"secret_text","name":"REDIRECT_TO","text":"'${url}'"}]}' > metadata.json
        break
    else
        if [ $i -eq 10 ]; then
            screen -X -S ngrok quit
            echo "die"
        else
            echo "wait 1s"; sleep 1
        fi
    fi
done
```

其中输出到 metadata.json 的是更新 Floudflare Workers 需要用到的文件，里面 REDIRECT_TO 的值就是要重定向的链接。  
Workers 有比较方便的 KV 用于存储，但是属于付费项目，所以我剑走偏锋地使用了 -> Workers 全局变量。  
下一步就是对比和提交的过程，代码中*用中文标记的内容是需要根据你自己的配置改动的地方*。  

``` shell
old_url=`cat url.txt`
if [ "$url" == "$old_url" ]; then
    echo "no new url is good url"
else
    echo "found new url, prepare to update cloudflare workers"
    # 更新 CF Workers
    update_result=`curl -s -X PUT "https://api.cloudflare.com/client/v4/accounts/帐号挨地/workers/scripts/卧壳名称" \
                        -H "Authorization: Bearer 授权信息" \
                        -F "metadata=@metadata.json;type=application/json" \
                        -F "script=@script.js;type=application/javascript"`
    success_str=`echo "$update_result" | jq -r .success`
    if [ $success_str == "true" ]; then
        echo $url > url.txt
    else
        echo "fail to update cloudflare workers"
    fi
fi
```

还需要的文件就是 script.js，里面套用一个 Workers 官方给出的 Hello World 模板即可，需要改动的部分就是返回的页面 head 部分需要设置自动跳转，跳转的地址就是上面设置的全局变量 REDIRECT_TO。需要注意的是得告诉浏览器这个网页不建议缓存，因为要跳转的地址可能经常变化。

``` javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})
const html = `<!DOCTYPE html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <meta HTTP-EQUIV='pragma' CONTENT='no-cache'>
    <meta HTTP-EQUIV='Cache-Control' CONTENT='no-store, must-revalidate'>
    <meta HTTP-EQUIV='expires' CONTENT='Wed, 26 Feb 1997 08:21:57 GMT'>
    <meta HTTP-EQUIV='expires' CONTENT='0'>
    <meta http-equiv='refresh' content='0;url=${REDIRECT_TO}'>
    <title>BACK GARDEN</title>
  </head>
  <body>
    正在跳转...  
  </body>
</html>
`
async function handleRequest(request) {
  return new Response(html, {headers:{'Content-Type': 'text/html'}})
}
```

## 缺点

这套方案仅适合测试或者个人使用的小项目，因为首次运行 Workers 有编译部署的短暂延迟，并且需要先加载网页才能跳转。并且 ngrok 速度也不快，总体上不太流畅，有那么点 lag。并且，地址会显示 ngrok 的子域名。  
所以还是建议测试好了应用，部署在 GCP 那种成熟的平台上，有免费配额，使用体验好很多。  

## 此外

我也尝试使用 Cloudflare 的 Page Rule，通过 302 跳转来达到目的，并且也能通过 api 来配置，可是解析没有成功，不知具体原因。如果有成功的小伙伴还请不吝赐教。  

## 次日凌晨增补

后来查看文档发现，Workers 可以直接 302 跳转，使用一下模板即可：  

``` javascript
async function handleRequest(request) {
  return Response.redirect(someURLToRedirectTo, code)
}
addEventListener('fetch', async event => {
  event.respondWith(handleRequest(event.request))
})

const someURLToRedirectTo = REDIRECT_TO
const code = 302
```

这样延迟又小了一些  
