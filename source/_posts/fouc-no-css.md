---
title: 通过代码读懂apply,call,bind及JS原型链
date: 2017-03-31 19:31:00
tags:
- apply,call,bind的作用
- _proto_
- 原型链
categories: 基础理论，原理
description: 
ref: 
---

## 原型链

### 使用new原型对象生成实例对象的缺点（原型链要解决的问题）

用原型对象生成实例对象，有一个缺点，那就是无法共享属性和方法。

例如，在Person对象的构造函数中，设置一个实例对象的共有属性species。
```javascript
　　function Person(name){
　　　　this.name = name;
　　　　this.species = '黄种人';
　　}
```
然后，生成两个实例对象：

```javascript
    var personA = new Person('张三');
　　var personB = new Person('Tom');
```

这两个对象的species属性是独立的，修改其中一个，不会影响到另一个。

```javascript
　　personA.species = '黑人';
    alert(personA.species); // 显示"黑人"
　　alert(personB.species); // 显示"黄种人"，不受personA的影响
```

每一个实例对象，都有自己的属性和方法的副本。这不仅无法做到数据共享，也是极大的资源浪费。

### prototype属性（原型链）的引入

在了在来自同一原型对象的实例对象间共享数据（对象），于是在原型对象中引入了prototype属性。该属性包含一个对象（以下简称"prototype对象"），所有实例对象间需要共享的属性和方法，都放在这个对象里面；那些不需要共享的属性和方法，就放在构造函数里面。
实例对象一旦创建，将自动引用prototype对象的属性和方法。也就是说，实例对象的属性和方法，分成两种，一种是本地的，另一种是引用的。
还是以Person为例，现在用prototype属性进行改写：
```javascript
　　function Person(name){
　　　　this.name = name;
　　}
　　Person.prototype = { species : '黄种人' };

　　var personA = new Person('张三');
　　var personB = new Person('Tom');

　　alert(personA.species); // 黄种人
　　alert(personB.species); // 黄种人
```

现在，species属性放在prototype对象里，是两个实例对象共享的。只要修改了prototype对象，就会同时影响到两个实例对象。
```javascript
　　Person.prototype.species = '黑人';

　　alert(personA.species); // 黑人
　　alert(personB.species); // 黑人
```

### 总结
由于源于同一原型对象的所有实例对象共享同一个prototype对象，那么从外界看起来，prototype对象就好像是实例对象的原型，而实例对象则好像"继承"了prototype对象一样。
这就是Javascript原型链的由来，也是Javascript继承机制的核心思想。

## apply,call及bind的使用
apply,call,bind的作用：改变某个函数运行时的上下文（context）环境，换句话说，就是为了改变函数体内部 this 指向。

调用格式：
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2]);
var newFunc = func.bind(this);

从上面可以看到：call和apply的作用相同，区别在于调用时传递的参数（apply第二个参数是数组格式）。bind的作用和call，apply相同，只是调用bind时不是立即执行func函数，而是返回新的函数对象，供后续代码调用。

看懂以下示例，几个方法的作用及区别就很明白了：

```
var func = function(arg1, arg2) {
    console.log("name="+this.name+",arg1="+arg1+",arg2="+arg2);
};

//将func函数内的this指向第一个参数，即{name:'li'}；后面2个参数为函数本身参数；
func.call({name:'li'},12,22)  //输出：name=li,arg1=12,arg2=22

//将func函数内的this指向第一个参数，即{name:'wang'}；后面2个参数为函数本身参数；
func.apply({name:'wang'},[33,44]) //输出：name=wang,arg1=33,arg2=44

//将func函数内的this指向第一个参数，即{name:'liu'}；，然后返回新的方法，供后续调用。
var newFunc = func.bind({name:'liu'});  
newFunc (55,66) //输出：name=liu,arg1=55,arg2=66
```