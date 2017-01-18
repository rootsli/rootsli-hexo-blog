---
title: 前端开发最佳实践之HTML5 Server-Sent Events串流的使用
date: 2017-01-17 20:31:16
tags:
- HTML5 串流
- SSE
- Server-Sent Events
- WebSocket
categories: 前端开发最佳实践
description: 文章介绍了HTML5 Server-Sent Events功能的使用以及利用此功能实现服务器消息的主动推送。
ref: https://blog.gtwang.org/web-development/stream-updates-with-server-sent-events/
---
## 简介
在传统的B/S架构中，当浏览器关心的某个数据发生变更时，服务器无法主动向浏览器端推送变更信息，只能由浏览器端通过AJAX定时轮询方式进行数据的查询和更新，这种方式下，数据的更新存在延迟，实时性不好。后来，出现的WebSocket彻底解决了该问题，但是使用起来较为复杂，而且需要服务器端的支持。
不过除了WebSocket，在HTML5标准中，还有一个Server-Sent Events（下文简称SSE）也可以实现服务器向浏览器主动推送消息。SSE的基本概念跟WebSocket有点类似，浏览器可以利用SSE来“订阅”服务器上的一个数据源，当这个数据源的数据有更新的时候，就会通过这条“订阅”的线路，
将数据主动推送给浏览器，从而实现数据的实时更新。

## SSE VS WebSocket
- **SSE**
    - 单向通信，只能服务器端往浏览器端推送数据，因此，比较适合股市行情，即时新闻等数据单向传输（服务端推浏览器端），数据量少的应用
    - 架构于传统的HTTP协议之上，向前兼容
    - 实现简单，服务器端只需添加几行应用级代码即可实现
    - 存在一些WebSocket没有的优点，例如：断线重连，事件ID与传送任意事件等
    - IE浏览器不支持（可以通过引入开源库解决）

- **WebSocket**
    - 全双工双向通信，功能强大
    - 无法向前兼容，需服务器端特别支持
    - 适合线上游戏，聊天程序及各种需要及时双向传输数据的应用
    - 无论是服务器端还是浏览器端实现都相对复杂

## SSE 浏览器兼容性
所有主流浏览器均支持SSE，除了IE。

```
提示：对于不支持Server-Sent Events功能的浏览器，建议使用开源库解决，例如：https://github.com/EventSource/eventsource
```

## SSE 浏览器端实现
- **step1.浏览器支持检测**
```
if (!!window.EventSource) {
  var source = new EventSource('stream.php');
} else {
  // 瀏覽器不支援 SSE
}
```

注意：当指定完整URL时，该URL必须与网页地址一致，不支持跨域，即不能为第三方服务地址。

- **step2.设置事件监听回调**
```
//数据接收回调
source.addEventListener('message', function(e) {
  console.log(e.data); //当服务端数据推送过来时，可以通过e.data获取数据
}, false);

source.addEventListener('open', function(e) {
  // 通信链路已建立
}, false);

source.addEventListener('error', function(e) {
  if (e.readyState == EventSource.CLOSED) {
    // 通信链路已关闭
  }
}, false);
```

SSE很好的一点是如果链路因为某些原因中断了，它会自动在大约3秒钟后重新连接，开发者也可以自己设置这个等待事件。

## SSE 服务器端实现

以下为Node.js的实现版本：

```
var http = require('http');
var sys = require('sys');
var fs = require('fs');

http.createServer(function(req, res) {
  //debugHeaders(req);

  if (req.headers.accept && req.headers.accept == 'text/event-stream') {
    if (req.url == '/events') {
      sendSSE(req, res);
    } else {
      res.writeHead(404);
      res.end();
    }
  } else {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.write(fs.readFileSync(__dirname + '/sse-node.html'));
    res.end();
  }
}).listen(8000);

function sendSSE(req, res) {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });

  var id = (new Date()).toLocaleTimeString();

  // Sends a SSE every 5 seconds on a single connection.
  setInterval(function() {
    constructSSE(res, id, (new Date()).toLocaleTimeString());
  }, 5000);

  constructSSE(res, id, (new Date()).toLocaleTimeString());
}

function constructSSE(res, id, data) {
  res.write('id: ' + id + '\n');
  res.write("data: " + data + '\n\n');
}

function debugHeaders(req) {
  sys.puts('URL: ' + req.url);
  for (var key in req.headers) {
    sys.puts(key + ': ' + req.headers[key]);
  }
  sys.puts('\n\n');
}
```

## 数据格式
使用SSE传输数据，需要符合它约定的格式。
基本格式：以 data: 开头，加上数据内容，最后以2个换行符 \n\n 结束，例如：

```
data: My message\n\n
```

如果要传输的数据量比较大的话，也可以将数据进行分行传输，每行都以data:开头，然后以一个换行符\n结尾（最后一行需要2个换行符），例如：

```
data: first line\n
data: second line\n\n
```

像上面这样连续的data行会被认为是一个数据，这些数据被传送到浏览器端时，只会触发一次数据接收回调。并且数据会被自动合并。上面这个例子，浏览器收到的数据为:
e.data = "first line\nsecond line"

- 传送JSON
如果要传送 JSON 格式的资料，可以这样写：

```
data: {\n
data: "msg": "hello",\n
data: "id": 123\n
data: }\n\n
```

浏览器端收到这个 JSON 数据后，可以这样处理：

```
source.addEventListener('message', function(e) {
  var data = JSON.parse(e.data);
  console.log(data.id, data.msg);
}, false);
```

- 事件分类

发送的数据可以根据事件名称，对数据进行分类。事件名称通过event来指定，浏览器在接收到含有event的数据后，就可以根据不同的事件名称来进行处理。

下面发送的数据中，包含三个不同的事件，分别为：普通消息事件（message），用户登录事件（userlogon）和更新事件（update）。

```
data: {"msg": "First message"}\n\n
event: userlogon\n
data: {"username": "John123"}\n\n
event: update\n
data: {"username": "John123", "emotion": "happy"}\n\n
```

浏览器端针对上面事件的处理代码：

```
source.addEventListener('message', function(e) {
  var data = JSON.parse(e.data);
  console.log(data.msg);
}, false);

source.addEventListener('userlogon', function(e) {
  var data = JSON.parse(e.data);
  console.log('User login:' + data.username);
}, false);

source.addEventListener('update', function(e) {
  var data = JSON.parse(e.data);
  console.log(data.username + ' is now ' + data.emotion);
}, false);
```

## 安全性

为了避免恶意攻击，在使用SSE接收数据时，需要检查数据中的e.origin是否为合法来源。如下：

```
source.addEventListener('message', function(e) {
  if (e.origin != 'http://www.data.com') {
    alert('Origin was not http://www.data.com');
    return;
  }
  // ...
}, false);
```
