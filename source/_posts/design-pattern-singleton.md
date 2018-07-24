---
title: JavaScript设计模式之单例模式
date: 2018-07-24 14:39:30
tags:
- 设计模式
categories:
- JavaScript
---

## 前言

系统开发中，单例模式的使用频率是比较高的，如皮肤，全局缓存等。系统设计中，单例模式可以减少系统内存的开销，合理的使用单例模式是系统优化的重要方法。

现在有一个构想，将系统的一组API封装，使得全局可以访问操作，需要注意的是，API具有push、shift等方法，会对全局的API数组进行操纵，全局的API对于系统而言是唯一的。

<!-- More -->

基于以上构想，我们可以通过单例模式的方法来实现API的操作对象。

## 方法一

通过getInstance方法取得唯一实例

```javascript
var ApiHandler = function () {
  this.apis = [];
  this.instance = null;
}
ApiHandler.prototype.push = function () {
  // ...
}
/* 其他方法略 */

/**
 * 提供getInstance函数，生成唯一单例
 */
ApiHandler.getInstance = function () {
  return this.instance ? this.instance : this.instance = new ApiHandler();
}

/**
 * 通过闭包和自调用函数实现，屏蔽this下的instance属性
 */
 ApiHandler.getInstance = (function(){
   var instance = null;
   return function () {
     return instance ? instance : instance = new ApiHandler();
   }
 })();

 var handler = ApiHandler.getInstance();
```

这种方法实现起来并不困难，也容易理解，但是值得注意的是必须给构造函数提供一个getInstance方法，在系统开发中，很有可能出现你无法控制的构造函数，这个时候很难保证你需要的构造函数存在getInstance方法.

另外一方面要生成一个单例对象，必须通过显式地调用getInstance方法，显然，我们更希望地是通过`new`关键字来生成一个单例对象。

## 方法二

通过闭包和自调用函数实现对构造函数包装，进一步实现单例模式，其实就是高阶函数的应用。

另外说一点，JavaScript的设计模式中很多都可以通过函数式编程来实现。

```javascript
var ApiHandler = (function () {
  var instance = null;
  var handler = function () {
    if (instance) {
      return instance;
    }
    this.apis = [];
    instance = this;
  }
  handler.prototype.push = function () {
    // ...
  }
  return handler;
})();

var a= new ApiHandler()
```

这种方法暴露出来的只有ApiHandler这个方法或者构造函数，只需要通过`new`关键字即可获得`handler`的唯一实例对象，进一步访问他的方法和属性，但通过这种方法，我们获得的实例的类型将会是`handler`，对于用户而言`handler`是未知的。

虽然这种方法满足了我们通过`new`关键字生成单例的期望，但对于已经存在的类或构造函数，还是无能为力。

## 方法三

对于方法一而言，它确实能满足一些场景的应用，但是对于没法重构的类或构造函数，这种方法就会失去用武之地

基于方法一中闭包和自调用函数的实现，我们可以单独实现一个代理类，通过代理类构造单例对象。

```javascript
var ProxySingleApiHandlre = (function () {
  var instance = null;
  return function () {
    return instance ? instance : instance = new ApiHandler();
  }
})();

var a = new ProxySingleApiHandlre();
// 或者
var b = ProxySingleApiHandlre();
```

通过代理实现单例模式，我们可以自己实现代理函数，不需要考虑目标类是否有提供唯一实例对象的接口了。

值得注意的是，对于JavaScript的构造函数，如果我们显式地返回一个对象，再通过`new`关键字，我们得到的并不是构造函数的克隆，而是返回的对象。

## 小结

JavaScript这门语言再设计之除是没有类的概念，即便是ES6的`class`和`extends`也只是对原型继承的封装，对于JavaScript而言，获得一个单例对象很简单，只要

```javascript
var obj = {};
```
或者

```javascript
window.obj = {};
```

所以对我们而言，要得到一个单例对象并不需要太多复杂的操作，只需要直接使用即可。不要受到其他语言的影响，js就是js，是不一样的烟火﹋o﹋ 。

## 参考

《JavaScript设计模式与开发实践》一书中对该模式有很详细的介绍，本文只是一个简单的说明，如果想要深入了解的同学可以去看原书。

[JavaScript设计模式与开发实践](http://www.ituring.com.cn/book/1632#)
