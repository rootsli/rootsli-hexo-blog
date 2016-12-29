---
title: 前端开发最佳实践之CORS请求在浏览器端的处理过程
date: 2016-12-28 17:58:04
tags:
- HTTP OPTIONS
- preflight request
- CORS
categories: 前端开发最佳实践
description: 关于http options请求的最佳实践介绍。了解本节内容，可以帮助开发者在coding中涉及CORS跨域请求时，避免由于请求参数错误或多余导致冗余请求的情况，从而使代码效率更高。
ref: http://www.ruanyifeng.com/blog/2016/04/cors.html
---
## CORS请求
CORS全称是"跨域资源共享"（Cross-origin resource sharing）。它允许浏览器向跨域服务器发送HTTP请求，从而克服了AJAX只能同源使用的限制。CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。对于前端开发者来说，发出一个CORS请求与普通请求并无差别，代码处理逻辑也大致一样，只要服务器实现了CORS接口，就可以跨域通信。但是对于浏览器来说，针对不同的CORS请求（浏览器把它分为简单请求和非简单请求），处理方式却不同，浏览器有可能在满足特定条件的请求头中添加一些附加头信息或者多出一次OPTIONS请求等。

## CORS跨域中的简单请求和非简单请求
浏览器将CORS请求分为2类：简单请求和非简单请求。并针对这2种请求类型的处理也有差别``。

### 简单请求需要满足以下2个件：
- **请求方法必须为以下方法之一：HEAD,GET,POST**
- **只能包含以下HTTP头信息：**
    - Accept
    - Accept-Language
    - Content-Language
    - Last-Event-ID
    - Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

### 非简单请求：
凡是不属于简单请求的CORS请求都属于非简单请求。例如：请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json等。

### 浏览器针对两种请求的处理过程
- **共同点**

无论是哪种CORS请求，浏览器在默认情况下都不会在请求中附带cookie和HTTP认证信息。除非同时满足以下2个条件：
- 服务器在options的应答头中设置：
```
Access-Control-Allow-Credentials: true
```
- 开发者必须在AJAX请求中打开withCredentials属性
```
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```

需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。

- **不同点**

不同点在于浏览器在处理非简单的CORS请求时，会先向服务器发送一个预检请求（preflight request，即OPTIONS请求）。这个请求的主要目的是想获知以下信息：
```
当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。
```

只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

- **简单CORS请求过程**

 （1）请求
```
GET /graph/sdp/one?duration=6h&endpoint=flexweb-wx-32&counter=cpu.busy HTTP/1.1
Host: query.falcon.sdp:9966
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Accept: application/json, text/javascript, */*; q=0.01
Origin: http://127.0.0.1:8081
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36
Referer: http://127.0.0.1:8081/modules/web/detail.html
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
```

可以看到简单跨域请求和普通非跨域请求基本一致，只是未发送cookie信息。

 （2）应答
    
```
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: Authorization,DNT,X-Mx-ReqToken,Keep-Alive,User-Agen,x-requested-with,Content-Type,Content-Length
Access-Control-Allow-Methods: POST,GET,OPTIONS,PUT
Access-Control-Allow-Origin: *
Content-Type: application/json; charset=UTF-8
Date: Wed, 28 Dec 2016 12:05:20 GMT
Transfer-Encoding: chunked
```

应答则比普通非跨域应答多出了4个特殊字段：
```
    Access-Control-Allow-Credentials
    Access-Control-Allow-Headers
    Access-Control-Allow-Methods
    Access-Control-Allow-Origin
```

- **非简单CORS请求过程**

step1.发送OPTIONS请求

（1）请求
```
OPTIONS /v0.2/auth/user/authority?path=webComponentApp_detail&catalog=detail.bts&type=USER_AUTHORITY HTTP/1.1
Host: sdp-portal-server.pre1.web.nd
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Access-Control-Request-Method: GET
Origin: http://127.0.0.1:8081
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36
Access-Control-Request-Headers: authorization, content-type
Accept: */*
Referer: http://127.0.0.1:8081/modules/web/detail.html
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
```

可以看到，携带了2个特殊字段：
```
Access-Control-Request-Method
Access-Control-Request-Headers
```

（2）应答

```
HTTP/1.1 200 OK
Server: nginx/1.10.0
Date: Wed, 28 Dec 2016 12:05:18 GMT
Content-Length: 0
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, HEAD, OPTIONS, PUT, DELETE, TRACE, PATCH
Access-Control-Allow-Headers: Origin, Accept, X-Requested-With, Content-Type, Access-Control-Request-Method, Access-Control-Request-Headers, Authorization, Cache-control, Orgname, vorg
Access-Control-Max-Age: 1800
Allow: GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH
```

step2.发送GET请求

```
GET /v0.2/environment/app/54fe96fe45ceeb7b8e1fed2e?type=ENVIRONMENT_LIST HTTP/1.1
Host: sdp-portal-server.pre1.web.nd
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Accept: application/json, text/javascript, */*; q=0.01
Origin: http://127.0.0.1:8081
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36
Authorization: MAC id="FDASFDSAFASD34214324321RFDSAFDSFFSA",nonce="4321432143214321:zl5cdxxxx",mac="FDSAFDSAFDSAFDSA+D9qsCal1I+tsWAZXtPfl07uI="
Content-Type: application/json
Referer: http://127.0.0.1:8081/modules/web/detail.html
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
```
应答略...

## 其他跨域方案
- jsonp
    - 利用 `<script>` 标签的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。

```
JSONP VS CORS
CORS与JSONP的使用目的相同，但是比JSONP更强大。
JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。
```

- postMessage

    - postMessage()方法允许来自不同源的脚本采用异步方式进行有限的通信，可以实现跨文本档、多窗口、跨域消息传递。  

- websocket

    - WebSocket实现了全双工通信，使WEB上的真正的实时通信成为可能。浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。  

- SSE

    - Server-Sent Events(SSE)功能，允许服务端推送数据到客户端(通常叫数据推送)。已经在浏览器上普遍支持，然而IE和Edge全系列不支持。

- ServiceWorker

    - 一个 service worker 是一段运行在浏览器后台进程里的脚本，它独立于当前页面，提供了那些不需要与web页面交互的功能在网页背后悄悄执行的能力。在将来，基于它可以实现消息推送，静默更新以及地理围栏等服务，但是目前它首先要具备的功能是拦截和处理网络请求，包括可编程的响应缓存管理。简而言之，这是让浏览器具有服务器功能的API，有了他还跨域个球球。不过这只是实验中API目前只有chrome和firfox高版本浏览器支持。   