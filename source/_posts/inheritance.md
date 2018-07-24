---
title: JavaScript之继承
date: 2018-07-24 18:04:01
tags:
- inheritance
categories:
- JavaScript
---

JavaScript是一门面向对象的语言，自然会提供继承的方法，但是由于JavaScript的特殊性，他的继承与其他面向对象的语言又有所不同，他提供更富有表现力的方式实现继承，但这看起来会与以其他的基于面向对象设计的语言有很大不同。

<!-- more  -->

## 所谓实例化

JavaScript可以说是一门基于原型设计的面向对象的语言，但是却留给我们一个非常困惑的关键字

```javascript
var a = new Constructor();
```
`new`经常会让我们忽略了JavaScript面向对象的本质，让我们无法察觉到他与其他面向对象设计的语言有何不同，实际上`new`关键字等同于如下操作，

```javascript

function _new_ () {
  var obj = {};
  var C = [].shift.call(arguments);
  obj.__proto__ = C.prototype;
  const ret = C.apply(obj, arguments);
  return typeof ret !== 'object' ? obj : ret;
}

```

## 函数化

在《JavaScript语言精粹》一书中，作者给出了一种十分优秀且实用的继承方法————函数化继承。

首先我们实现一个函数化的构造器。

```javascript
var animal = function(spec, my) {
  var that = {};
  spec = deepClone(spec); // 避免外部不完全的编码直接修改spec
  var eat = function () {
    // code
  };
  var saying = function () {
    return 'animal:' + spec.say;
  }
  that.eat = eat;
  that.saying = saying;
  return that;
}
```

备注：不要试图以其他高级语言的相似性去理解这种继承方法，尽管你可以做到。

让我们来构造小猫，他继承自`animal`。

```javascript
var cat = function (spec) {
  var that = animal(spec); // 实现继承
  var getName = function () {
    return spec.name || 'unknown';
  }
  that.getName = getName;
  return that;
}

// 家里新来一只小甜甜，会叫`喵喵喵`

var sugar = cat({ name: '小甜甜', say: '喵喵喵' });

// 介绍一下
sugar.saying();
sugar.getName();
```

函数化继承实现了单继承的模式，现在又有一个问题，当我们重写父类方法的同时，如何在子类中访问父类呢。《JavaScript语言精粹》作者给出了一种实现。

```javascript
Object.prototype.superior = function (name) {
  var that = this, method = that[name];
  return function () {
    return method.apply(that, arguments);
  }
}

var cat = function (spec) {
  var that = animal(spec); // 实现继承
  var superSaying = that.superior('saying');
  var getName = function () {
    return spec.name || 'unknown';
  }
  var saying = function(){
    console.log(spec.name + ':', superSaying());
  }
  that.getName = getName;
  that.saying = saying;
  return that;
}
var sugar = cat({ name: '小甜甜', say: '喵喵喵' });
sugar.saying();
sugar.getName();
```
