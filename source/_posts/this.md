---
title: JavaScript之this指向
date: 2018-07-24 13:30:46
tags:
- this
categories:
- JavaScript
---

## 前言

对于JavaScript的this，我们需要的知道的是：

> this对象是在函数执行时基于函数的运行环境动态绑定的。

这一点对于我们理解this对象的指向尤为重要，也是这一特性，导致同一个函数可能由于你调用方式的不同，this对象的指向也发生改变。比如

<!-- More -->

```javascript
var name = 'jack';
var person = {
  name: 'bob',
  getName: function () {
    return this.name;
  }
}

// 方式一
console.log(person.getName());

// 方式二
var getName = person.getName;
console.log(getName());
```

执行一下上边的代码，我们会发现，两次打印的值并不一样。同一个函数，由于执行方法的不同，导致得到不同的结果。。

在不考虑箭头函数的情况下，一般函数的调用我们可以分为四种情况。


## 作为对象的方法被调用

当函数作为对象的方法被调用时，this一般指向对象本身。

```javascript
var person = {
  name: 'bob',
  getName: function () {
    console.log(this);
    return this.name;
  }
};
person.getName();

var obj = {
  name: 'obj',
  obj1: {
    name: 'obj1',
    obj2: {
      name: 'obj2',
      getName: function () {
        console.log(this);
        return this.name;
      }
    }
  }
}
obj.obj1.obj2.getName();
```

需要注意的是，当多层嵌套时，this指向他的直接调用者，本例中指向`obj2`

## 作为普通函数被调用

js天生适合函数式编程，我们可以编写一个函数，让他四处传递，作为普通函数被直接调用的情况，在JavaScript中尤为常见。此时this对象的指向为全局对象。

```javascript
var func1 = function () {
  console.log('fucn1',this);
};
function func2() {
  console.log('func2',this);
}

func1();

func2();

setTimeout(function(){console.log('settimeout',this)});

(function () {
  console.log('立即执行函数',this);
  var func3 = function () {
    console.log('func3', this);
  }
  func3();
})();

var obj = {
  _func: function () {
    var func5 = function () {
      console.log('func5', this);
    };
    func5();
  }
}
obj._func();
```

执行上述代码发现，所有打印的this都是指向window全局对象的（运行环境为浏览器），作为普通函数被调用时，函数的this对象均指向全局对象。

`func1`和`func2`不用多说，属于函数最基本的调用形式。对于setTimeout而言，这里给出的例子是一个匿名的回调函数，他的内部会将这个回调函数作为普通函数直接执行，所以也属于这种情况。立即执行函数的效果其实和`func1`或`func2`一样，只是写法上更加简洁，而内部的`func3`，也是作为普通函数被调用的。

到这里就可以解答前言中给出的两个例子：

```javascript
var name = 'jack';
var person = {
  name: 'bob',
  getName: function () {
    return this.name;
  }
}

// 方式一
console.log(person.getName());

// 方式二
var getName = person.getName;
console.log(getName());
```

对于方式一而言，没有疑问，函数作为对象的调用者指向对象本身。第二种情况此时函数被`getName`引用，即使申明时函数是作为对象属性的，但是this指向只能由执行时的上下文环境决定，故方式二中`getName`函数是作为普通函数被执行，返回的是全局的`name`，即`jack`。

## 作为构造函数被调用

new关键字在JavaScript中只类似原型链的语法糖，通过new关键字会生成一个新的对象，而此时的this指向的就是即将出生的对象。

```javascript
var Person = function (name, age) {
  this.name = name;
  this.age = age;
}

var jack = new Person('jack', 10);
```

this指向新生成的对象，即`jack`。

## apply、call和bind的调用

很多时候，我们需要明确的this指向，甚至动态的改变或者绑定this对象。`apply`、`call`和`bind`都可以动态绑定this对象。

```javascript
var name = 'jack';
var person = {
  name: 'bob',
  getName: function () {
    return this.name;
  }
}

// 方式一

var getName = person.getName;
console.log(getName.apply(person)); // 'bob'

// 方式二
console.log(person.getName.call()); // 'jack'，默认绑定全局对象
```

**特别**

在使用bind方法时，需要注意调用bind方法返回的函数，this对象将不会被apply、call和bind的改变。

```javascript
var job = { name: 'job' };

var bob = { nameL: 'bob' };

var getName = function () { return this.name };

var getJobName = getName.bind(job);

console.log(getJobName()); // 'job'

var getBobName =  getJobName.bind(bob);

console.log(getBobName()); // 'job'

```

[更多关于apply、call和bind的用法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

## 箭头函数

箭头函数是ES6新增的规范，主流浏览器基本都已支持，写法上较之普通函数更加简单（函数式编程不能更爽，可惜我不太会），除此之外，比以前普通函数`执行时绑定this`相比，箭头函数内部是没有this对象的。

举个例子

```javascript
var jack = {
  age: 21,
  getAge: () => {
    return this.age;
  }
}

var age = 23;
var getAge = jack.getAge;

console.log(jack.getAge()); // ???
console.log(getAge()); // ???
```

执行以上函数，发现打印的结果都是`23`，我相信很多同学都以为应该是`21`（我一开始也是这么想的，现实狠狠地给了我一个大耳巴子）。

箭头函数本身没有提供this对象，它使用的this来自于申明时的外部环境，而普通的如`jack`这样的对象，实际上也是没有this对象的。那么`getAge`使用的this对象就只能是全局对象了，故结果都是`23`。

总结一下，确定箭头函数内部使用的this对象时，找他申明环境中的this对象即可。

## 小结

到这里大部分关于this对象的问题，基本都能解决了。但JavaScript变化万千，谁又知道还有多少坑呢！
