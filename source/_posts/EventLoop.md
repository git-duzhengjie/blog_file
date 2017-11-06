---
title: EventLoop
date: 2017-10-20 07:58:16
tags: [javascript, event_loop, node]
---
[原文地址](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
## 单线程语言Javascript

JavaScript之所以是单线程语言，是和它的用途有关的；作为浏览器脚本语言，它的主要用途就是与用户互动，操作DOM；如果Javascript是多线程的，多个线程同时操作一个dom或增添，或修改，或删除，那浏览器到底以谁为准？所以，单线程成了该语言的核心特征；但是为了利用多核CPU的计算能力，H5提出了Web Worker标准，允许javascript创建多个子线程，但是子线程完全受主线程控制，且不得操作DOM；

<!--more-->

## 任务队列

单线程就意味着，所有操作只能一个一个完成，为了避免阻塞操作浪费大量的时间，javascript中将执行任务分为两种，一种同步任务，另一种是异步任务；
同步任务就是指：在主线程上执行的任务，只有前一个执行完毕，后一个才能执行；
异步任务指：不进入主线程执行而进入任务队列执行的任务，只有任务队列通知某个异步任务可以执行了的时候，该任务才会进入主线程进行执行；
所以，js的执行方式如下：
1. 所有的同步任务都在主线程上执行，形成一个执行栈；
2. 主线程之外，还有一个任务队列，只要异步任务有了执行结果，就会在任务队列中放置一个事件；
3. 一旦执行栈中的任务执行完毕，就会读取任务队列，看看里面有哪些事件，那些事件对应的异步任务结束等待状态，进入执行栈，开始执行；
4. 主线程不断重复上面的第三步；
下面就是主线程和任务队列的示意图：
![EventLoop](http://image.beekka.com/blog/2014/bg2014100801.jpg)

## 事件和回调函数

任务队列也可以理解为消息队列，IO设备完成一项任务，就会在任务队列中添加一个事件，表示相关的异步任务可以进入执行栈了；
回调函数，就是那些会被主线程挂起的代码，异步任务必须指定回调函数，主线程开始执行异步任务，就是执行相应的回调函数；

## Event Loop 图解

![EventLoop](http://image.beekka.com/blog/2014/bg2014100802.png)
图中：主线程运行的时候，产生堆和栈栈中的代码调用各种外部的API，它们在任务队列中加入各种事件，只要栈中的代码执行完毕，主线程就会读取任务队列，依次执行事件所对应的回调函数；

## 定时器

setTimeot(fn, 0)指主线程在得了空闲时间马上执行fn，尽可能早的执行；它在任务队列尾部添加一个事件，等到同步任务和r任务队列现有的事件都处理完了，才会执行fn；所以，setTimeout只是将事件插入了任务队列并不一定会按照设定的时间执行；
H5规定第二个参数不得小于4毫秒，低于这个值，就会自动增加；对于DOM变动会涉及页面重新渲染的部分，不会立即执行，而是每16毫秒执行一次；

## Node.js中的Event Loop

Node.js也是单线程的，但是它的机制和浏览器不一样，如下图：
![node event loop](http://image.beekka.com/blog/2014/bg2014100803.png)
1. V8引擎解析Javascript脚本；
2. 解析后的代码调用Node API；
3. [libuv](https://github.com/libuv/libuv)库负责Node API的执行，它将不同的任务分给不同的线程，形成一个Event Loop，以异步的形式将执行结果返回给V8引擎；
4. V8引擎再将结果返回给用户；
除了setTimeout、setInterval，Node.js还提供了另外两个与任务队列有关的函数process.nextTick和setImmediate