---
title: 前端开发最佳实践之CSS3 pointer-events:none
date: 2017-01-09 20:12:17
tags:
- CSS3
- pointer-events
categories: 前端开发最佳实践
description: 文章介绍了如何利用CSS3的pointer-events:none属性，完全屏蔽超链接，按钮等的鼠标事件及效果。
ref: http://www.ruanyifeng.com/blog/2016/04/cors.html
---
## 表单提交中的按钮禁用
在做富客户端的互联网Web应用时，通常都涉及到表单提交。如果希望做得稍微完美些的话，通常都要实现以下效果：表单项未填写或者输入数据非法时，提交按钮处于禁用状态（即鼠标放到提交按钮上，无任何反应）。当输入数据合法时，提交按钮变为可用状态。如下图：

![image](/assets/img/disable.png)

![image](/assets/img/enable.png)

 ```
 问题：在如上场景中，如果我们只切换CSS样式，使按钮“看上去”像是不可用的。但是实际上，当鼠标放到按钮上或是点击鼠标时，事件仍然被执行（仍然响应提交事件）。
 ```

## 解决之道
想要不写一行javascript代码，就做到对鼠标事件（包括鼠标效果）的完全禁用，那只能是CSS3的pointer-events方能做到了。通过对按钮元素设置CSS3属性：
 ```
 pointer-events: none;
 ```
 即可禁用该元素下注册的所有事件。
 
 ## 浏览器兼容性
 
 ![image](/assets/img/pinters-envent.png)
