---
title: 关于事件循环、task
date: 2018-10-17 22:20:13
tags: 
- js
---

# 关于事件循环、task

## 事件循环

**首先，JS是单线程的，是基于事件循环的。**

### 1. 为什么JavaScript是单线程？

JavaScript语言的一大特点就是单线程，也就是说，同一个时间只能做一件事

作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

### 2. 任务队列

**问题：**

单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长（比如ajax网络请求），后一个任务就不得不一直等着。

**方案：**

于是，所有任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。

* 同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；
* 异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

**浏览器中异步执行的运行机制：**

1. 所有同步任务都在主线程上面执行，形成一个[执行栈](http://www.ruanyifeng.com/blog/2013/11/stack.html)(execution context stack)。
2. 主线程之外，还存在一个“任务队列”（task queue）。只要异步任务有了运行结果，就在“任务队列”之中放置一个事件。
3. 一旦“执行栈”中的所有同步任务执行完毕，系统就会读取“任务队列”，检查里面是否有事件。那些对应的异步任务(回调函数)，于是结束等待状态，进入执行栈，开始执行。
4. 主线程不断重复上面的第三步。

**主线程与任务队列示意图**

![主线程与任务队列示意图](http://www.ruanyifeng.com/blogimg/asset/2014/bg2014100801.jpg)

### 3. 事件和回调函数


**任务队列：**

一个事件的队列（也可以理解成消息的队列），IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。

主线程读取"任务队列"，就是读取里面有哪些事件。

**任务队列中的事件**

除了IO设备的事件以外，还包括一些用户产生的事件（比如鼠标点击、页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取。

**回调函数**

"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

注意：

* "任务队列"是一个先进先出的数据结构，排在前面的事件，优先被主线程读取。主线程的读取过程基本上是自动的，只要执行栈一清空，"任务队列"上第一位的事件就自动进入主线程。

* 但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。


### 4.event loop

**事件循环示意图**

![事件循环与任务队列](https://user-gold-cdn.xitu.io/2018/3/25/1625bc965301b755?imageslim)

只要主线程空了，就会去读取“任务队列”，这就是JavaScript的运行机制。这个过程会不断重复。

上图中，主线程运行的时候，产生堆（heap）和栈（stack），栈中的代码调用各种外部API，它们在"任务队列"中加入各种事件（click，load，done）。只要栈中的代码执行完毕，主线程就会去读取"任务队列"，依次执行那些事件所对应的回调函数。

## task

实际上，js引擎并不只维护一个任务队列，总共会有两种任务。

1. 宏任务 - Task(macroTask)
	* setTimeout
	* setInterval
	* setImmediate
	* I/O
	* UI rendering

2. 微任务 - microTask
	* Promise 
	* Object.observe
	* MutationObserver
	* process.nextTick（node环境）

那么，宏任务和微任务在执行上有什么不一样呢？

### 试一试

```bash
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```

正确的顺序应该是：

```
script start

script end

promise1

promise2

setTimeout
```
但是因为不同浏览器的不同版本的支持情况不尽相同，具体顺序会有点差别。

注意：引用文章写于2015年，列举的浏览器版本为firefox 40的阶段（现在最新版本为62），原文提到的火狐浏览器怪异问题，目前测试是不存在了。故原因一段不翻译，感兴趣可以自行阅读。

### 关于microtask的行为

1. microtask 会令某段代码成为异步任务，而且该任务会在下一个task（即macrotask）执行前执行。
2. 满足任一条件时，microtask的队列就会执行：
	* 在回调函数（注意是回调函数不是task，只要是挂载在事件上的回调函数就可以）执行后，只要没有主线程中没有执行中的js代码（可理解为栈中没有执行环境）
	* 每当有一个macrotask执行完毕
3. 在执行microtask的过程中，如果又添加了新的microtask，该microtask将会添加到当前microtask queue的最后并且会执行

> promise也是遵循这个规则的。

### 结论

1. task（macrotask）的执行是有顺序的，而且浏览器有可能会在两个任务执行之间进行渲染
2. microtask的执行也是有顺序的。只要满足任一条件就会执行microtask队列：
	* 在每个回调函数（注意是回调函数不是task，只要是挂载在事件上的回调函数就可以）执行后，只要没有其他js代码执行中（可理解为栈中没有执行环境）
	* 在每个macrotask结束时

> refer2 中，有更复杂的例子，可以学习

---

20181108分享后：

1. 重新分析示例，结论表述调整
2. 补充以下例子：

```
// 添加三个 Task
// Task 1
setTimeout(function() {
  console.log(4);
}, 0);

// Task 2
setTimeout(function() {
  console.log(6);
  // 添加 microTask
  promise.then(function() {
    console.log(8);
  });
}, 0);

// Task 3
setTimeout(function() {
  console.log(7);
}, 0);

var promise = new Promise(function executor(resolve) {
  console.log(1);
  for (var i = 0; i < 10000; i++) {
    i == 9999 && resolve();
  }
  console.log(2);
}).then(function() {
  console.log(5);
});

console.log(3);
```

> 执行结果：1 2 3 5 4 6 8 7

至此，算是比较完整理解了event loop和task了。

---

todo：总结node中的event loop和相关api

refer：

1. [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
2. [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
3. https://juejin.im/post/5baf37835188255c6c624d38