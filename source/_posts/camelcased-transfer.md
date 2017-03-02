---
title: 一个将JSON对象连字符式转为驼峰式的处理函数
date: 2017-03-02 20:31:51
tags:
- hasOwnProperty用法
- JSON字段连字符式转为驼峰式
- 连字符转为驼峰式
categories: 方法函数，技巧
description: 一个将JSON数据属性由连字符式转换为驼峰式的方法。
---

 ## 起因
 最近，服务端在进行接口改版，原来旧接口返回的JSON数据使用的是驼峰式命名法，格式如下：
 ```
 buildType : 2
 chineseName : "部署测试"
 createTime : 1472489455000
 creator : 339691
 demandAnalysisUrl : "应用描述33"
 desc : "应用描述22"
 designAnalysisUrl : "应用描述44"
 members : [
   {
    yourName : 'li',
    yourAge : 23
   }
 ]
 ```
现在新改版的v2.0接口，由于基于新的框架，JSON数据使用的是连字符命名法，格式如下：
 ```
 build_type : 2
 chinese_name : "部署测试"
 create_time : 1472489455000
 creator : 339691
 demand_analysisUrl : "应用描述33"
 desc : "应用描述22"
 design_analysisUrl : "应用描述44"
 members : [
   {
    your_name : 'li',
    your_nge : 23
   }
 ]
 ```
 
因此，为了使改动量最小，需要实现一个适配函数，新接口返回结果经过该函数处理后，得到驼峰式命名的JSON数据，对该函数的要求如下：

- 支持将一个JSON对象的连字符命名变量转换为驼峰式命名
- 支持对对象进行深度递归遍历

## 函数实现

```
/**
 * 将对象的变量名由连字符式转为驼峰式，支持对象的深度遍历转换
 * @param obj JSON对象
 * @return JSON 驼峰式的JSON对象
 */
function obj2CamelCased(obj){
    if (!(obj instanceof Object)) {
        return obj;
    }

    var newObj = {};
    for (var prop in obj) {
        if (obj.hasOwnProperty(prop)) { //推荐在 for in 时，总是使用 hasOwnProperty 进行判断，没人可以保证运行的代码环境是否被污染过。 
            var camelProp = prop.replace(/_([a-z])/g, function (g) {
                return g[1].toUpperCase();
            });
            if (obj[prop] instanceof Array) { //值为数组的处理
                newObj[camelProp] = [];
                var oldArray = obj[prop];
                for (var k = 0, kLen = oldArray.length; k < kLen; k++) {
                    newObj[camelProp].push(arguments.callee(oldArray[k]));
                }
            } else if (obj[prop] instanceof Object) { //值为对象的处理
                newObj[camelProp] = arguments.callee(obj[prop]);
            } else { //值为字符串，或数字等的处理
                newObj[camelProp] = obj[prop];
            }
        }
    }
    return newObj;
}

```