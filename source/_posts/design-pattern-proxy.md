---
title: JavaScript设计模式之代理模式
date: 2018-07-24 14:35:38
tags:
- 设计模式
categories:
- JavaScript
---
## 前言

代理模式是为一个对象提供代用品，进一步控制对象访问的一种设计模式。代理模式在JavaScript语言中出现的频率比较高，比如代理API，代理工厂等。

下边，我将以两个简单的例子演示代理模式。（注：自编实例，不合理之处欢迎指出）

<!-- More -->

## 代理单例

在之前的博文中，我已经讲过[单例模式](./design-pattern-singleton.md)了，这里提一下使用代理实现单例的方式。

```javascript
var SingleObject = function () {
  this.handler = null;
  /* ...some code... */
}

// 提供一个单例的代理函数

var proxyFactory = function (v_interface) {
  return (function () {
    var instance = null;
    return function () {
      return instance ? instance : instance = new v_interface();
    }
  })()
}

/**
 * 运用curry的思想，得到实现SingleObject单例的函数
 */
var getSingleObject = proxyFactory(SingleObject);
var getOtherObject = proxyFactory(OtherObject);

var singleton = getSingleObject();
var otherSingle = getOtherObject();

```

较之前单例的实现方法，引入了curry的思想，实现了代理单例工厂的复用。这里给curry打个问号，因为尽管最后返回了一个工厂函数，但proxyFactory函数其实只接受一个参数，在这里仅当做高阶函数看待吧。

**我个人比较喜欢通过代理的方式实现单例**

## 图片加载

可能由于网络或者图片尺寸的影响，我们会在触发请求之后很长一段时间（可能几秒甚至更长）才能拿到图片资源，中途我们可以给图片预设一个loading的动图。

```javascript
var ProxyImage = function (img) {
  var proxyImg = new Image();
  proxyImg.onload = function () {
    img.src = this.src;
  };
  return {
    setSrc: function (src) {
      img.src = './loading.gif';
      proxyImg.src = src;
    }
  }
}

ProxyImage(document.getElementsByTagName('img')[0]).setSrc('...path');
```

通过 proxyImg 代理图片的加载过程，当加载成功后，通知真实的img设置src。

## 小结

代理模式就到这里了，实际工作中，代理的应用非常广泛，如代理HTTP请求，代理模拟数据等。有时间再补充吧。
