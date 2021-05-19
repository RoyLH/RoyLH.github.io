---
title: EventLoop
tags:
  - Node.js
categories:
  - Node.js
abbrlink: 34574
date: 2021-05-19 15:11:26
---

![](https://image.fundebug.com/2019-01-14-004.png)

## Event Loop是什么

---

event loop是一个执行模型，在不同的地方有不同的实现。浏览器和NodeJS基于不同的技术实现了各自的Event Loop。
- 浏览器的Event Loop是在html5的规范中明确定义。
- NodeJS的Event Loop是基于libuv实现的。可以参考Node的官方文档以及libuv的官方文档。
- libuv已经对Event Loop做出了实现，而HTML5规范中只是定义了浏览器中Event Loop的模型，具体的实现留给了浏览器厂商。

---

## 宏队列和微队列

---

宏队列，macrotask，也叫tasks。 一些异步任务的回调会依次进入macro task queue，等待后续被调用，这些异步任务包括：

- setTimeout
- setInterval
- setImmediate (Node独有)
- requestAnimationFrame (浏览器独有)
- I/O
- UI rendering (浏览器独有)

微队列，microtask，也叫jobs。 另一些异步任务的回调会依次进入micro task queue，等待后续被调用，这些异步任务包括：

- process.nextTick (Node独有)
- Promise
- Object.observe
- MutationObserver
（注：这里只针对浏览器和NodeJS）

---

## 
---

浏览器的Event Loop
![](https://user-gold-cdn.xitu.io/2018/9/5/165a8667bb6e623e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
这张图将浏览器的Event Loop完整的描述了出来，我来讲执行一个JavaScript代码的具体流程：

执行全局Script同步代码，这些同步代码有一些是同步语句，有一些是异步语句（比如setTimeout等）；
全局Script代码执行完毕后，调用栈Stack会清空；
从微队列microtask queue中取出位于队首的回调任务，放入调用栈Stack中执行，执行完后microtask queue长度减1；
继续取出位于队首的任务，放入调用栈Stack中执行，以此类推，直到直到把microtask queue中的所有任务都执行完毕。注意，如果在执行microtask的过程中，又产生了microtask，那么会加入到队列的末尾，也会在这个周期被调用执行；
microtask queue中的所有任务都执行完毕，此时microtask queue为空队列，调用栈Stack也为空；
取出宏队列macrotask queue中位于队首的任务，放入Stack中执行；
执行完毕后，调用栈Stack为空；
重复第3-7个步骤；
重复第3-7个步骤；
......

这里归纳3个重点：

宏队列macrotask一次只从队列中取一个任务执行，执行完后就去执行微任务队列中的任务；
微任务队列中所有的任务都会被依次取出来执行，知道microtask queue为空；
图中没有画UI rendering的节点，因为这个是由浏览器自行判断决定的，但是只要执行UI rendering，它的节点是在执行完所有的microtask之后，下一个macrotask之前，紧跟着执行UI render。

---