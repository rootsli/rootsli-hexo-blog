---
title: 关于使用vue-loader时，热部署无法使用问题
date: 2017-01-24 20:35:08
tags:
- vue-cli
- vue hot reload
categories: 一些坑
description: 在使用vue-loader的热部署时遇到的一个坑，这里记录下。
ref: https://www.zhihu.com/question/38834718
---
 ## 问题原因及解决方法
 在使用vue-loader的热部署时，修改源码文件，界面没有自动更新。查了一堆资料，百度，google，花了几个小时，终于解决了。原来不是插件的问题，问题出在使用的编辑器上。 由于webstorm，它默认保存在临时文件，把settings=>appearance=>system=>synchornization=>最后一项勾去掉，热部署替换功能就能正常使用了。
 
 ![image](/assets/img/hot-reload.png)
