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

## 类继承

JavaScript可以说是一门基于原型设计的面向对象的语言，但是却留给我们一个非常困惑的继承方法——类继承

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

可以说原型链是实现类继承的关键了。下边我们利用类继承实现狗继承动物

```javascript
var Animal = function () {
  this.name = 'animal'
}

var Dog = function () {
  this.name = 'dog';
}

Dog.prototype = new Animal(); // 继承Animal

Dog.prototype.constructor = Dog; // 还原构造函数

var dog = new Dog();


dog instanceof Dog; // true

dog instanceof Animal; // true

```

我们可以适当优化下

```javascript
var _extends_  = function (Child, Parent) {
  if (!Parent) return Child;
  if (/* 类型判断 */) throw new TypeError('');

  var F = function () {}; //构造空函数，减少内存开销

  F.prototype = Parent.prototype;

  Child.prototype = new F();

  Child.prototype.constructor = Child;

}

```

## 原型继承

JavaScript是无类的，也是弱类型，理论上来说只要给对象或他的原型上赋予某个属性或方法，他就能实现特定的功能，原型继承就是基于这一点

得益于ES的发展，我们有了简洁的方法实现这一功能。

```javascript

var parent = {
  name: 'some',
  getName: function () {
    return this.name;
  }
}

var child = Object.create(parent);

child.name = 'jack';

child.getName();

```

当然，并不是所有浏览器都支持`Object.create`方法，但实现基于原型的继承却并不那么难。

```javascript
var _create_ = function (p) {
  if(/* 类型校验 */) throw new TypeError('');

  var F = function () {};

  F.prototype = p;

  return new F();
}

```

## 拷贝继承

基于上一种的实现，我们有新的继承思路，对象上的属性拷贝，举个栗子。

```javascript
var clone = function (p, c = {}) {
  /* 容错处理 ... */

  return { ...p, ...c };
}

var parent = {
  para: '属性',
  getP: function () {
    return this.para;
  }
}

var child = {
  para: '自己的属性',
}

child = clone(parent, child);

child.getP();
```

以上就实现了一个简单但破绽百出的克隆拷贝的例子，JavaScript的灵活性让我们有更多的方式是实现你想要的功能，这里我就不举例了。有兴趣的同学可以自己试试。

## 函数化继承

在《JavaScript语言精粹》一书中，作者给出了一种十分优秀且实用的继承方法——函数化继承。

首先我们实现一个函数化的构造器。

```javascript
var animal = function(spec, my) {
  var that = {};
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

函数化继承实现了单继承的模式，现在又有一个问题，当我们重写父类方法的同时，如何在子类中访问父类呢。《JavaScript语言精粹》作者给出了一种实现，也可以实现自己的super方法。

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
