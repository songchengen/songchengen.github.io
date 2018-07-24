---
title: JavaScript设计模式之发布订阅模式
date: 2018-07-24 14:39:47
tags:
- 设计模式
categories:
- JavaScript
---

引用[维基百科](https://zh.wikipedia.org/wiki/%E5%8F%91%E5%B8%83/%E8%AE%A2%E9%98%85)的一段话

> 在软件架构中，发布-订阅是一种消息范式，消息的发送者（称为发布者）不会将消息直接发送给特定的接收者（称为订阅者）。而是将发布的消息分为不同的类别，无需了解哪些订阅者（如果有的话）可能存在。同样的，订阅者可以表达对一个或多个类别的兴趣，只接收感兴趣的消息，无需了解哪些发布者（如果有的话）存在。

<!-- more -->


## 前言

Node中的事件模块——`events`，让我们可以对一个事件主体设置监听事件（订阅），当监听事件被发出（发布）时，监听者或者订阅会受到请求，触发回调。

```javascript
// 监听
events.on('something', (data) => {
  // ...some options
});

// 发出
events.emit('something', data);

```

使用发布订阅模式，可以很好的把代码解耦，在接触ES6/7之前，一直是被我认为是处理异步的利器，即使是与Promise、Generator等新规范相比。处理异步之外，发布订阅模式也有更多的市场。

## 异步

假定我们现在有一个需求，需要有序的向服务器的a、b、c三个接口请求。我们可以通过回调嵌套实现，如下。

```javascript
ajax(a, () => {
  ajax(b, () => {
    ajax(c, () => { /* code */ }))
  })
})
```

似乎还能接受，但是当接口数增加到5个、6个甚至更多呢，代码会变得非常难以维护，请求之间耦合度太大。如果使用发布订阅模式，每一个请求只需要在成功之后发出自己的事件，依赖方订阅事件就好了，各个接口完全不需要关心彼此的实现逻辑，同时也避免了回调地狱。如下

```javascript
var events = new EventEmitter();

/* b接口订阅a事件，以下类似 */
events.on('a', () => { ajaxB() })

events.on('b', () => { ajaxC() })

events.on('c', () => { /* code */ })

const ajaxA = () => ajax(a, events.emit('a'));

const ajaxB = () => ajax(b, events.emit('b'));

const ajaxC = () => ajax(c, events.emit('c'));

ajaxA();

```

如果有一天，b接口请求完成之后，需要处理一些同步业务。你只需要增加一行订阅代码，引入业务代码即可，如下。

```javascript
events.on('b', service());
```

下面我们来实现一个EventEmmiter
```javascript
const events = {
  eventsSet: {},
  // 订阅事件时，加入事件集合
  on: function (key, fn) {
    this.eventsSet[key] = this.eventsSet[key] || [];
    this.eventsSet[key].push(fn);
  },
  emit: function(key) {
    const args = [].slice.call(arguments, 1);
    const me = this;
    if (this.eventsSet[key]) {

      // 如果事件源被订阅，则执行订阅者的逻辑代码
      // 当订阅者执行普通函数时，apply绑定生效，this指向事件主体
      this.eventsSet[key].forEach(f => f.apply(me, args))
    }
  }
}

const EventEmitter = function () {}
EventEmitter.prototype = events;

```

## 组件通信

用过React的同学应该知道，组件之间的数据传输是单向的，但有的时候又需要子组件向父组件发送数据，怎么办？一开始想到的是redux，后来发现子组件是通用的功能组件，可以提出来复用，这时候再用redux似乎不是那么理想。还好我们有发布订阅模式。

基于上个实例中的`EventEmitter`实现子组件向父组件通信。

注：并非一定要通过prop传递。

```javascript
// 父组件
const Parent = ({ data }) => {

  const event = new EventEmitter();

  const onMessage = (message) => {
    // ...code 修改Store
  }

  event.on('message', (message) => {
    onMessage(message);
  })
  return (
    <div>
      <Child event={event} data={data}/>
    </div>
  )
}
// 子组件
const Child = ({ event, data }) => {

  const clickBtn = () => {
    event.emit('message', '修改后的数据')
  }

  return (
    <div>
      <button onClick={clickBtn}>点我</button>
      {{ data }}
    </div>
  )
}
```

**扩展**

data修改后势必会触发组件的重渲染，每一次重新渲染都要实例化一个EventEmitter，这其实是不必要的开销。修改方案如下。

* 增加单例工厂，避免每次重渲染之后EventEmitter的实例化（也可以不用纯组件，在组件类的构造方法中实例化）。

* 单例之后引发的新问题：之前保留的订阅还未被取消。可以通过扩展EventEmitter方法，实现取消订阅或者限制订阅者数量的方法。

实现如下。

```javascript
const events = {
  eventsSet: {},
  // 订阅事件时，加入事件集合
  on: function (key, fn) {
    this.eventsSet[key] = this.eventsSet[key] || [];
    this.eventsSet[key].push(fn);
    return true;
  },
  emit: function(key) {
    const args = [].slice.call(arguments, 1);
    const me = this;
    if (this.eventsSet[key]) {

      // 如果事件源被订阅，则执行订阅者的逻辑代码
      // 当订阅者执行普通函数时，apply绑定生效，this指向事件主体
      this.eventsSet[key].forEach(f => f.apply(me, args))
    }
  },
  // 之前的订阅将全部取消，仅保存本次订阅
  only: function (key, fn) {
    this.eventsSet[key] = [fn];
  },
  // 移除remove之前的订阅
  remove: functin (key, fn) {
    if(! this.eventsSet[key]) return;
    if (!fn) {
      this.eventsSet[key] = undefined;
      return;
    }
    this.eventsSet[key] = this.eventsSet[key].filter(i => i !== fn);
  }
}
```

## 有一个大胆想法

写Node的时候路由规划简直不能太烦，我就想能不能通过发布订阅模式实现路由的动态注入 :smirk: ，下面是我做的部分尝试（以下操作，生产坏境慎用）

```javascript

const fs = require('fs');
const { join } = require('path');

const EventRouter = {};

/**
 * router set
 * @type {{}}
 */
EventRouter.eventsSet = {};

/**
 * register router
 * @param {String} key router method and path
 * @param {Function} fn controller
 */
EventRouter.on = function (key, fn) {
  // 同一个路由，仅允许订阅一次
  if (this.eventsSet[key]) throw new Error(`The same route [${key}] cannot be registered multiple times`);
  this.eventsSet[key] = fn;
  return true;
}.bind(EventRouter); // 强行绑定事件对象，避免书写习惯造成的异常

/**
 * call controller
 * @param {String} key
 * @param {KoaContent} ctx
 * @param {KoaNext} next
 */
EventRouter.emit = async function (key, ctx, next) {
  if (!this.eventsSet[key]) { // 未订阅路由，手动跳出
    await next();
  } else { // 否则执行controller，同时注入ctx和next，并绑定app（非箭头函数绑定生效，否则this对象指向声明所在上下文）
    this.eventsSet[key].call(ctx.app, ctx, next);
  }
}.bind(EventRouter);

/**
 * Automatic scanning
 * @param {String} path
 * @param {RegExp} filename
 */
EventRouter.run = function ({ path, filename = /\.controller\.(js|ts)$/ }) {
  if (!path) throw new Error('path is not be empty');

  if (!(filename instanceof RegExp)) throw new TypeError(`filename must be RegExp, but get ${typeof filename}`);

  log('cyan', 'Waiting register routers...');
  // 递归扫描文件，自动导入订阅路由
  const scanDir = (scanPath, matchReEexp) => {
    fs.readdirSync(scanPath).forEach((f) => {
      const fpath = join(scanPath, f);
      if (fs.statSync(fpath).isDirectory()) {
        scanDir(fpath, matchReEexp);
      } else if ((matchReEexp && matchReEexp.test(f)) || /\.(js|ts)$/.test(f)) {
        require(fpath);
        log('yellow', fpath);
      }
    });
  };

  scanDir(path, filename);

  log('green', 'Register success');
};


module.exports = EventRouter;

// 入口

const EventRouter = require('./EventRouter');

// 将router事件挂载到全局环境
global.router = global.router || EventRouter;

module.exports = (config) => {
  // 扫描文件，订阅路由
  router.run(config);
  return async (ctx, next) => {
    // 发布路由事件，有path和method组成唯一事件源
    router.emit(`${ctx.method.toLowerCase()} ${ctx.path}`, ctx, next);
  };
};

```

[源码](https://github.com/songchengen/koa2-event-router)

## 总结

发布订阅模式在JavaScript的实践中，极为便利，Vue.js的数据绑定渲染就是基于类似的实现（观察者模式？？？），此模式也是我使用较为频繁的模式（React实现数据共享，见例二）
