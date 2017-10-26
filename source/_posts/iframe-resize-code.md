---
title: 一段iframe自适应代码的分享
date: 2017-10-26 22:31:16
tags:
- iframe resize
categories: 前端开发最佳实践
description: 
---

代码如下:

``` javascript
(function () {
  window.iframeResizePostMessage = function (name) {
    if (window.parent) {
      var height = document.documentElement.scrollHeight || document.body.scrollHeight;
      var width = document.documentElement.scrollWidth || document.body.scrollWidth;
      window.parent.postMessage(JSON.stringify({ 'type': 'iframe-resize', name: name || '', height: height, width: width }), '*'); // name各业务组件自己定义
    }
  }
})();
```