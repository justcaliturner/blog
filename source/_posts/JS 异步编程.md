---
title: JS 异步编程
date: 2022-04-27 01:03:00
---

最近因项目需要，使用了后端 Node.js 相关的一些工具，对于一直写前端的我来说遇到了一些没怎么接触过的基础知识，比如异步编程，虽然这是种编程方法，不区分前后端环境，但对我这个前端小白来说，之前确实接触的不多，这里记录一些学习笔记。

##### 多线程和单线程
- 多线程：程序可在多个线程中同时执行多个任务
- 单线程：程序在单线程中仅可依次执行单个任务

##### 同步和异步
- 同步：任务的执行顺序和排列顺序相同，多个任务会在主线程中逐个依次执行，下一个任务必须等待上一个任务完成后才可执行，优点是执行起来逻辑清晰明了，缺点是若上一个任务执行较久则会发生线程阻塞的情况。

- 异步：任务的执行顺序和排列顺序可不同，上一个任务正在执行等待结果的同时，下一个任务即可被执行，优点是不容易阻塞线程，缺点是多任务执行顺序交错复杂，难以预估。
![This is an image](./images/1.jpg)

##### JavaScript 的异步实现
JavaScript 引擎是单线程的，故在其主线程中只能进行同步编程，无法实现异步编程，但浏览器环境和 Node.js 环境是多线程的，比如浏览器内有：GUI 线程、JS 引擎线程、定时器线程、事件触发线程、异步 http 请求线程，所以某些时候 JS 可以通过与环境内的其他线程进行通讯，以实现异步编程的效果，总而言之，JavaScript 的异步实现本质上是在解决如何让 JavaScript 的主线程与环境内的其他线程进行通讯，其原理就是通过一个中间桥梁 “消息队列” 让主线程与其他线程进行交互。

![This is an image](./images/2.jpg)

##### 异步实现之事件轮询
![This is an image](./images/3.jpg)
- 事件轮询 Event loop
    > 一个为实现 JS 并发模型的执行机制，负责执行、收集、处理事件及其子任务，简单讲是为了实现异步执行的一种机制，其本质是不断重复从消息队列中取出消息并执行的一种过程。
    - 栈 Stack
    > 函数调用形成的一个由多个帧组成的栈。
    - 堆 Heap
    > 对象被分配在堆中，一个用来表示一大块（通常是非结构化）的内存区域。
    - 消息队列 Message queue
    > JavaScript 运行时的一个待处理消息的消息队列，每一个消息都关联着一个用以处理这个消息的回调函数，其逻辑是先进先出。

##### 事件轮询中异步任务的执行逻辑
异步任务分为宏任务和微任务，事件轮询中会先执行宏任务，再执行微任务，依次循环往复。
![This is an image](./images/4.jpg)
- 异步任务 Asynchronous task
    - 宏任务 Macro task
        - script 
        - setTimeout/setInterval
        - setlmmediate
        - I/O
        - UI rendering
    - 微任务 Micro task
        - Promise
        - Object.observe
        - MutationObserver
        - postMessage

##### 举例：`setTimeout`
- `setTimeout` 是一个异步的宏任务，当它被调用时，会先将回调方法移出此次执行，等下一次事件轮询时检查是否到了指定时间，如果到了，执行回调方法，如果没到，则等待下一次事件轮询的检查。
- 其中计时任务会在浏览器的定时器线程中执行，因为主线程可能会阻塞导致计时不准确，而具体到浏览器的运行环境，最终都会调用到操作系统的定时器。
- 因无法确定 `setTimeout` 调用前是否还有待执行的同步任务，所以 `setTimeout` 本质上无法确定开始执行的时间，只能确定结束时间和持续时长。

##### 异步任务的控制
上文讲到多个异步任务的执行顺序交错复杂，难以预估，那么有效的控制其在上下文中的执行顺序就显得非常关键，这里就不得不提到 `Promise` 和 `async` `await` 的使用

##### `Promise`

`Promise` 对象用于表示一个异步任务的最终完成状态，有如下三种状态：
- 等待中（pending）: 表示还未得到执行结果。
- 已兑现（fulfilled）: 表示操作成功完成的结果。
- 已拒绝（rejected）: 表示操作失败的结果。

它的可以通过 `new Promise(executor)` 实例化一个 Promise 对象，它的作用是通过获取异步任务的三种不同状态来控制上下文任务的执行顺序：

```javascript
function demo1() {
    setTimeout(() => {
        console.log(1)
    }, 1000)
}
function demo2() {
    console.log(2)
}
demo1()
demo2()
// 以上代码的执行结果为 2, 1
// 因为 demo1 的 setTimeout 是异步任务，
// demo2 不会等 demo1 的结果出现再执行

function demo1() {
    return new Promise((resolve, reject) => {
        setTimeout(()=>{
            console.log(1)
            resolve()
        }, 1000)
    })
}
function demo2() {
    console.log(2)
}
demo1().then(demo2)
// 以上代码的执行结果为 1, 2
// 因为 demo2 需要等 demo1 的 Promise 实例的结果出现才会执行
```


###### 参数
该对象可传入一个类型为函数的参数，用于被构造函数执行，该函数可传入 2 个参数，它们的类型也是函数，第一个参数会在 Promise 实例已兑现时调用，第二个参数会在 Promise 实例已拒绝时调用：
```javascript
const demo = new Promise((resolve, reject)=>{
    // 其他代码
    resolve() // 已兑现时调用
    reject() // 已拒绝时调用
})
```

`resolve()` 可传入一个参数，作为 Promise 实例成功完成的执行结果，同样，`reject()` 可传入一个参数作为 Promise 实例错误时的结果：
```javascript
const demo1 = new Promise((resolve, reject)=>{
    const result = 'Success!'
    resolve(result) // result 将会作为执行成功的结果
})

const demo2 = new Promise((resolve, reject)=>{
    const error = new Error('failed')
    reject(error) // result 将会作为执行错误的结果
})
```

###### `Promise.then()`
`Promise.then()` 顾名思义就是当 Promise 实例完成后（执行错误或成功），继续执行其他代码的方法，它可传入一个类型为函数的参数作为继续执行的函数，该函数会接收一个参数，该参数是 Promise 实例的执行结果，它会作为参数传入该函数内。

```javascript
const demo = new Promise((resolve, reject)=>{
    const result = 'Success!'
    resolve(result)
})
demo.then((result)=>{
    console.log(result) // Success!
})

```

###### `Promise.all()`
`Promise.all()` 是等待所有的 Promise 实例都完成（或第一个失败）后调用的方法：

```javascript
const demo1 = Promise.resolve(1);
const demo2 = 2;
const demo3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 3);
});

Promise.all([demo1, demo2, demo3]).then((values) => {
  console.log(values); // [ 1, 2, 3 ]
});
```
###### 链式调用
`Promise.then()` 方法将进一步的操作与一个变为已敲定状态的 Promise 关联起来，这些方法还会返回一个新生成的 Promise 对象，这个对象可以被非强制性的用来做链式调用：
```javascript
const demo1 = function () {
    return new Promise((resolve, reject) => {
        setTimeout(()=>{
            console.log(1)
            resolve()
        }, 1000)
    })
}
const demo2 = function () {
    return new Promise((resolve, reject) => {
        console.log(2)
        resolve()
    })
}
const demo3 = function () {
    return new Promise((resolve, reject) => {
        console.log(3)
        resolve()
    })
}

demo1().then(demo2).then(demo3) // 1, 2, 3
```

<br>

##### `async` `await`
`async` 可声明一个函数为异步函数
`await` 操作符用于等待一个 Promise 对象敲定其状态, 它只能在异步函数 async function 内部使用，其简化了 Promise 的一些常规写法：

```javascript
async function foo() {
   await 1
}
```
以上代码等价于
```javascript
function foo() {
   return Promise.resolve(1).then(() => undefined)
}
```

如下案例简化了上文 `Promise` 中的最后一个案例
```javascript
const demo1 = function () {
    return new Promise((resolve, reject) => {
        setTimeout(()=>{
            console.log(1)
            resolve()
        }, 1000)
    })
}
const demo2 = function () {
    return new Promise((resolve, reject) => {
        console.log(2)
        resolve()
    })
}
const demo3 = function () {
    return new Promise((resolve, reject) => {
        console.log(3)
        resolve()
    })
}

function demo(){
    demo1()
    demo2()
    demo3()
}

async function asyncDemo(){
    await demo1()
    await demo2()
    await demo3()
}
demo() // 2, 3, 1
asyncDemo() // 1, 2, 3
```