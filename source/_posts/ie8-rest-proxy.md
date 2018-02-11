---
title: 前端开发最佳实践之IE8下解决CORS问题（REST proxy方案）
date: 2018-01-11 17:41:46
tags:
- REST proxy
- IE8兼容
categories: 前端开发最佳实践
description: 
---
## 方案解决的问题
- **适合于存在CORS跨域时，解决IE8下的兼容问题**
- **适合于一些对功能进行增强的场景，比如合并请求**

## 代理过程
- **方案由 5 个过程组成**
    - client: raw request -> wrapper request
    - server: wrapper request -> raw request
    - server: process raw request
    - server: raw response -> wrapper response
    - client: wrapper response -> raw response
    
    如下图所示：
    
    ![](/assets/img/rest-proxy.png)
    
- **wrapper-request**
    
    ```
    {
        "$url": "{url}",
        "$method":"{method}",
        "$headers":"{header object}",
        "$body":"{body object}"
    }
    ```
    
- **wrapper-response**
  ```
  {
      "$headers":"{header object}",
      "$body":"{body object}"
      "$status":"200",
      "$status_text":"{status text}"
  }
  ```
  
## 代理承载
- **可以依赖以下内容来承载请求代理的数据**
    - url	可以承载 method, url 的代理
    - headers	可以承载 method, url 的代理
    - body 可以所有的代理

- **可以依赖以下内容来承载响应代理的数据**
    - headers	可以承载 status、status_text
    - body 可以所有的代理

## 本地调试
我们开发的组件，都是通过iframe的方式被业务系统所引用。由于在IE下，iframe模拟接入，存在同源限制（即iframe的src地址不能和浏览器里的url地址相同）。
因此，在IE下调试，需要解决同源问题。我们使用的方式是，通过修改etc\hosts文件，将同一本地IP地址映射到不同的域名下，保证本地地址在IE下可调试。

## 支持特性
- **IE8/9的跨域访问支持**
    - 使用 XDomainRequest对象
    - 仅能使用GET及POST
    - 无法设置 request.headers
    - 无法读取 response.headers
    - 仅支持同一协议跨域，即不支持 HTTP 跨 HTTPS，反之亦然
- **方案**
使用 body 进行代理，所有的请求都转成 POST 请求，具体如下
  ```
  #REQUEST
  PUT /urser/100 HTTP/1.1
  Host: sdp.nd
  Accept: application/json
  Content-Type: application/json
   
  {
      "name":"myresource"
      "size":102323
  }
   
  #RESPONSE
  HTTP/1.1 200 OK
  Content-Type: application/json
  {
      "name":"myresource"
      "size":102323
  }
  ```
  
  代理后的为
  
  ```
  #REQUEST
  POST /urser/100?$proxy=body HTTP/1.1
  {
      "$method": "PUT",
      "$headers": {
          "Host": "sdp.nd",
          "Accept": "application/json",
          "Content-Type": "application/json"
      },
      "$body": {
          "name":"myresource"
          "size":102323
      }
  }
   
  #RESPONSE
  HTTP/1.1 200 OK
  Content-Type: application/json
  {
      "$status": 200,
      "$status_text": "OK",
      "$headers": {
          "Content-Type": "application/json"
      },
      "$body": {
          "name":"myresource"
          "size":102323
      }
  }
  ```
  
## 参考
[XDomainRequest – Restrictions, Limitations and Workarounds](https://blogs.msdn.microsoft.com/ieinternals/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds/)

