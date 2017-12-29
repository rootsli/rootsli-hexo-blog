---
title: 实现兼容ie8+的踩坑路程
author: 郑美双(917520)
date: 2017-11-27
tags:
- ie
- ie9
- ie8
- react
- 工程院三部一处

categories:
- 经验总结
---

> 最近收到需求抽奖Web版需要兼容ie8+的。于是开始踩坑的路程


## ie8，ie9首先考虑跨域问题

主要是采用服务端代理的方法，在项目代码里面放入java代码

（1）服务端支持在请求的接口加入前缀/v0.1/dispatcher，

（2）在请求的接口头部加入Dispatcher头

例如：

  ![image](/assets/img/cors.png)

  这样就可以通过访问同域的服务端转发接口，实现跨域请求


## 解决es3保留字的问题

保留字如：

```
default, class等

```

报错信息：

```
缺少标识符

```

解决方法：

在webpack1的配置文件中加入配置：

```
postLoaders: [{
            test: /\.js$/,
            loaders: ['es3ify-loader']
        }]

```


## 解决压缩后UglifyJsPlugin的问题

报错信息：

```
缺少标识符

```

解决方法：

在配置里面加入screw_ie8,压缩的时候不要去掉ie8的代码

 ```
 new webpack.optimize.UglifyJsPlugin({
            compress: {
                properties: false,
                warnings: false,
                screw_ie8: false
            },
            output: {
                beautify: true,
                comments: false,
                quote_keys: true,
                screw_ie8: false
            },
            mangle: {
                screw_ie8: false
            }
        })
```

## 上面改完后，继续报错

报错信息：

  ![image](/assets/img/error.png)

解决方法：

原先的webpack版本装的是1.15.2版本的，后面降为1.13.2版本就ok了

## 解决发生异常未捕获的错误

报错信息：

```
从core-js的_object-dp.js发出了错误：发生异常，未捕获

```

解决方法：

原先是使用babel-polyfill用来解决es6的api的兼容es5的问题。

现在因为core-js抛出错误了，所以直接废弃babel-polyfill

分别引入 (注： babel-polyfill是包含core-js和regnerator-runtime/runtime的)

```
import 'core-js-ie8'
import 'regenerator-runtime/runtime'

```
虽然core-js-ie8能完美解决这个问题，但是意外发现又引发了另外一个问题，core-js-ie8这个包实现的Object.assign在360和ie8+下是有问题的，于是乎就自己发了个npm包

core-js的issue也有相关的[core-js](https://github.com/zloirock/core-js/pull/258)

最终解决问题的代码变成：

```
import 'core-js-for-ie8'
import 'regenerator-runtime/runtime'

```
## 解决es5的API兼容的问题

解决方法：

在index.html的模板文件里面引入shim和sham：

```
<!--[if lt IE 9]>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/es5-shim/4.5.7/es5-shim.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/es5-shim/4.5.7/es5-sham.min.js"></script>
  <![endif]-->

```

## 继续报错，苦逼了

报错信息：

```
 无法修改属性type，不允许此命令

```

问题原因：

登录页面上有个“是否显示密码”；可以切换修改, 代码如下：

```
{ isShowPassword ? <input type="text" /> : <input type="password" /> }

```

因为对React来说，有虚拟Dom，对比下dom结构都没有改变，只是type改变，不会重新渲染新的input元素，只会做个diff，把type替换掉，于是报这个错了。

解决方法：

React组件只要带了key属性，都会重新render的，不会做diff比较。

于是最终更正后的代码：

```
{ isShowPassword ? <input type="text" key="text" /> : <input type="password" key="password" />

```

## 解决ie8下的事件不生效的问题：

问题：

```

<input type="text" onChange={this.onChange} onClick={this.onClick} onInput={this.onInput} onPropertyChange={this.onPropertyChange} />

<input type="checkbox" onChange={this.onChange} />

以上代码:
（1）<input type="checkbox"/>元素上的onChange事件是生效的

（2）<input type="text"/>元素上只有OnClick事件是生效的，其他都不生效

```

解决方法：


```
（1）保留onInut事件，其他事件不需要
<input type="text" onInput={this.onInput} />
（2）index.html的模板里面添加
<!--[if IE 8]><script src="//cdnjs.cloudflare.com/ajax/libs/ie8/0.3.2/ie8.js"></script><![endif]-->

```

## 解决Observable库的问题：

问题：

```
import { Observable } from 'rx-lite'

这个库是支持ie10+的，并不支持ie8，9

```
解决方法：

```
很开心的是，有个兼容库rx-lite-compat是支持ie8+的，心里默默开心，于是代码变成如下：

import { Observable } from 'rx-lite-compat'


```

## 上面解决完Observable问题的后遗症

问题根源：

```
开心的太早了，引入rx-lite-compat又报另外一个问题了，这库里面的事件报错了，估计不兼容了

```

报错信息：

![image](/assets/img/keycode.png)

```
无法获取未定义或null 引用的属性“keyCode”

```


解决方法：

```
引入ie8.js文件后，一切正常，好开心了。

<!--[if IE 8]><script src="//cdnjs.cloudflare.com/ajax/libs/ie8/0.3.2/ie8.js"></script><![endif]-->

```

## 样式兼容问题：

（1） transform:translate(-50%)这个是不支持的

（2） 透明度也是不支持的,改造后的代码：

```
 if (browserLessIE8()) {
            document.getElementById('shadow').style.backgroundColor = '#000'
            document.getElementById('shadow').style.filter = 'progid:DXImageTransform.Microsoft.Alpha(Opacity = 80)'
        } else {
            document.getElementById('shadow').style.background = 'rgba(0,0,0,0.8)'
        }

```

## 解决promise的兼容问题：

```
可以引入es6-promise库支持ie8，使用如下：

require('es6-promise').polyfill()

```

## 解决fetch的兼容问题：

问题根源：

```
 （1）发现请求成功了，也引入了es6-promise了，但是还是无法进入then方法，原来是fetch不支持ie8

（2）isomorphic-fetch是不支持ie8的

```

解决方法：

```
替换成universal-fetch支持ie8的

```

## console的polyfill：

问题原因：

```
 ie8没有打开控制台的时候，是没有console这个对象了，所以需要对console进行polyfil

```

解决方法：

```
import 'console-polyfill'

```

## 解决ie8下React项目在某些时候a标签无法跳转路由的情况

问题原因：

```
 <a href="#/home"><button>我来试一试</button></a>

 （1）以上代码在ie8下跳转失效

 （2）因为a标签里面包了button标签，从而导致跳转失败了

```

解决方法：

```

<a href="#/home"><div>我来试一试</div></a>

（1）把button标签改成div就可以了
（2）所以注意不要a标签里面包含button标签哦

```

# lodash版本：
lodash 从4.0.0开始支持的环境有： Chrome 46-47, Firefox 42-43, IE 9-11, Edge 13, Safari 8-9, Node.js 0.10.x, 0.12.x, 4.x, & 5.x, & PhantomJS 1.9.8。已不再支持IE6~IE8。


如果想兼容IE6~IE8，可以使用3.x版本。3.x版本支持的环境有：
Chrome 43-44, Firefox 38-39, IE 6-11, MS Edge, Safari 5-8, ChakraNode 0.12.2, io.js 2.5.0, Node.js 0.8.28, 0.10.40, & 0.12.7, PhantomJS 1.9.8, RingoJS 0.11, & Rhino1.7.6

lodash 3.x 版本没有直接提供可用的js， 需要手动构建。

```
安装
npm i -g lodash-cli@3.10.1
安装完成后执行
lodash compat
会输出兼容IE6~IE8的版本lodash.custom.js及lodash.custom.min.js
```


lodash 3.10.1版本文档地址： https://lodash.com/docs/3.10.1#template

# es6对象中使用取值器和赋值器报错

报错信息：

```
SCRIPT1003: 缺少 ':'

问题定位到：

var Auth = {

	  __access_token: null,

	  get accessToken() {
	    return this.__access_token;
	  },

	  set accessToken(val) {
	    this.__access_token = val;
	  },

	  __mac_key: null,

	  get macKey() {
	    return this.__mac_key;
	  },

	  set macKey(val) {
	    this.__mac_key = val;
	  }
}
get set后面是空格，缺少':'

```

问题原因：

```
babel转化的时候没有加入插件babel-plugin-transform-es5-property-mutators

```

解决方法：


```
.babelrc里面引入这个插件：

{
  "plugins": ["transform-es5-property-mutators"]
}
```

# 上面那个问题引发了另外一个问题：

上面的es6的get和set的代码，babel编译后的代码也会有get和set属性，编译后如下：


```
{
    key: 'accessToken',
    get: function get() {
      return this.__access_token;
    },
    set: function set(val) {
      this.__access_token = val;
    }
```

----------------------------


```
export { default as request } from './request'
export { default as RBAC } from './rbac'
export { default as RBACAdapt } from './rbac/instance'
```

上面这个代码也是类似的，bable编译后的代码也会有get和set属性如下：


```
Object.defineProperty(exports, 'RBACAdapt', {
  enumerable: true,
  get: function get() {
    return _interopRequireDefault(_instance)['default'];
  }
});
```

问题原因：


```
1. ie8不支持设置访问器属性，即便是引了es5-shim;
2. Babel 会把export xxx from ‘xx’ 语法转码为访问器属性设置的exports对象。
```


解决方法：

1. 代码里面不要使用get，set
2. export { default as request } from './request'类似这样的写法要分两句写：


```
import request from './request'

export request
```




# babel插件和webpack的兼容ie8的loader可以等价：


```
es3ify-loader相当于babelrc里面的两个插件
transform-es3-property-literals
transform-es3-member-expression-literal

webpack里面要是有用es3ify-loader,在.babelrc里面就不用配置
transform-es3-property-literals
transform-es3-member-expression-literal
这两个插件了

反之亦然
```

# 要支持ie8，注意的几个点：

1.webpack要用1.x，2.x不支持ie8了

2.源码中不要使用es6的get和set访问器

3.不要写成export x from 'x'，要分两步走

4.react要用0.14.x版本



## 参考资料：

[Webpack-IE低版本兼容指南](https://github.com/zuojj/fedlab/issues/5)

[IE8下使用React开发总结](http://blog.suzper.com/2017/04/17/IE8%E4%B8%8B%E4%BD%BF%E7%94%A8react%E6%80%BB%E7%BB%93/)

[defineProperty](https://github.com/reactjs/react-redux/issues/133)


[ie8.js](https://github.com/facebook/react/issues/3131)


[Exception thrown and not caught](https://github.com/xcatliu/react-ie8/issues/2)


[Make your React app work in IE8](http://react-ie8.xcatliu.com/)

[让Webpack+Babel支持IE8](https://www.maizhiying.me/posts/2017/03/01/webpack-babel-ie8-support.html)

[alibaba的兼容ie8的项目](http://www.aliued.com/?p=3240)
