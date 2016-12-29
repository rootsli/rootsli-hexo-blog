---
title: 前端开发最佳实践之CSS3 calc()函数
date: 2016-12-29 14:54:14
tags:
- CSS3
- calc()函数
- CSS计算属性
- 自适应布局
categories: 前端开发最佳实践
description: 在进行前端的自适应布局时，使用calc会带来很大的便利性，本篇文章主要介绍calc属性在自适应布局中的应用
ref: 
---

## 一个布局问题的优化
最近在做一个新的web运维系统，要求页面内容要能自适应屏幕视窗。如下图：( 图1)

![image](/assets/img/layouta.jpg)
 
有2个地方涉及自适应浏览器窗口：
* 左侧菜单栏部分（区域2和区域3），区域2的菜单内容为动态生成，区域3要求始终位于页面底部。
* 右侧主内容区域（区域4，区域5，区域6和区域7）。其中区域4高度固定，区域5的高度要随着浏览器视窗高度动态铺满。区域6，区域7要求也要动态铺满，且高度各位浏览器可视区域高度的1/2。

以上自适应部分，除非区域内部内容高度超出区域高度，否则不允许出现滚动条。

最初实现这个布局的是一位新同学，吧啦吧啦用了一堆js代码，通过监控窗口的resize事件总算完成了（大致实现方法应该大家都能想到，这里就不贴代码了）。由于框架通过路由来实现跳转，各个区域自然不在同一个js里面。导致代码看起来不那么“美观”，不好维护，而且在计算高度时，由于浏览器差异性，导致用户体验也不是很好。

于是问题出来了，有没有更好的方法实现呢？使这样的动态布局不需要依赖js来处理，从而与js解耦。答案就是CSS3的`calc()`方法了。于是又跟新人吧啦吧啦讲了一堆，最后仅使用`calc()`，没有写一行js，把这个布局问题完美解决了。


## calc()简介
 calc属于CSS3的一个函数，任何元素的宽高度都可以使用calc()函数进行计算；calc()函数支持 "+", "-", "*", "/" 运算符。
 浏览器支持情况如下：( 图2)
 
![](/assets/img/calc-compatibility.jpg)
 
calc()语法示例：

 ```
 width: calc(100% - 10px)；
 ```
 
 ## calc()典型应用场景
 
 当在一个纵向（或横向）排列的区域块中，其中某几个区域高度（或宽度）固定，另一些区域需要根据浏览器高度（或宽度）填满剩余空间时，建议使用calc()函数实现。如下图：（ 图3,页面整体布局参考图1）
 
![](/assets/img/calc-menu.jpg)


* 区域1和区域3高度固定，区域1紧贴顶部，区域3紧贴底部，区域2根据配置动态生成，并要求区域2高度能根据浏览器高度自适应剩余区域，且当区域2内部内容高度过高而出现滚动条时，滚动条上下滚动不会导致区域2与区域3重叠。
* 不满足要求（菜单栏滚动条出现后，区域2和区域3重叠）

![](/assets/img/calc-menu-scroll2.jpg)

* 满足要求（菜单栏滚动条出现后，区域2和区域3不重叠）

![](/assets/img/calc-menu-scroll1.jpg)

* 实现代码

CSS源码：

```
#portal-header {
    position: fixed;
    top: 0px;
    width: 100%;
    height: 70px;
    padding: 0px;
    background-color: #387AB7;
    z-index: 8000;
    min-width: 860px;
}

#portal-sidebar {
    position: fixed;
    top: 70px;
    width: 230px;
    background-color: #EAEDF1;
    z-index: 8000;
    overflow-y: auto;
    bottom: 0;
}
/* 区域2高度使用了calc()函数结合min-height来满足布局要求 */
#sidebar-scroll {
    min-height: -moz-calc(100% - 120px);
    min-height: -webkit-calc(100% - 120px);
    min-height: calc(100% - 120px);
}
.nav-feed-back {
    overflow: hidden;
    position: relative;
    height: 120px;
    padding: 16px;
    margin: auto;
    box-sizing: border-box;
}
```

HTML源码

```
<div id="portal-header">
    <!-- 区域1 -->
</div>
<nav id="portal-sidebar">
    <div id="sidebar-scroll">
        <!-- 区域2 -->
    </div>
    <div class="nav-feed-back">
        <!-- 区域3 -->
    </div>
</nav>
```

从代码可知，为了使区域2的高度能自适应浏览器高度，使用了calc()函数。
