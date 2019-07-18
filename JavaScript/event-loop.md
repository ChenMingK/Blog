>这里只介绍浏览器中的事件循环，node 中的事件循环放在 node.js 专题部分

## 灵魂三问
JavaScript 为什么是单线程的？JavaScript 为什么需要异步？JavaScript 单线程又是如何实现异步的？


**1.JavaScript 为什么是单线程的?**


现在有 2 个线程 process1 process2，假设 JavaScript 是多线程的，所以他们可以对同一个 dom 同时进行操作。process1 删除了该 dom，而 process2 编辑了该 dom，同时下达 2 个矛盾的命令，浏览器究竟该如何执行呢？这样想，JavaScript 为什么被设计成单线程应该就容易理解了吧。


**2.JavaScript 为什么需要异步?**


如果 JavaScript 中不存在异步，只能自上而下执行，如果上一行解析时间很长，那么下面的代码就会被阻塞。对于用户而言，阻塞就意味着"卡死"，这样就导致了很差的用户体验，所以 JavaScript 中存在异步执行。


**3.JavaScript 单线程又是如何实现异步的呢?**


是通过**事件循环**来实现的

>为了利用多核 CPU 的计算能力，HTML5 提出 Web Worker 标准，允许 JavaScript 脚本创建多个线程，但是子线程完全受主线程控制，且不得操作 DOM。所以，这个新标准并没有改变 JavaScript 单线程的本质。

为什么说 JavaScript 是“非阻塞”的语言？非阻塞即：“程序不会因为某个任务而停下来”
```js
console.log("程序时间：" + new Date().getTime())

setTimeout(function () {
     console.log("暂停一秒：" + new Date().getTime())
}, 1000)

console.log('这是暂停一秒之后的时间：' + new Date().getTime())
```
简单来，如果上图的 setTimeout 换成 C++ 的 sleep(1000) 之类的，那么 C++ 是会确实地“睡眠”那么段时间的，而 JS 不会。如果我就是想实现这个功能呢？可以利用 Promise 实现：
```js
async function test () {
  console.log('start')
  await sleep(3000)
  console.log('3s has passed')
}

function sleep (ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve()
    }, ms)
  })
}
```
## 任务队列
**任务队列（task queue）**
当遇到一个异步事件后，JavaScript 引擎并不会一直等待异步事件返回结果，而是会将这个事件挂在与执行栈不同的队列中，我们称之为任务队列。
这些任务又被细分为宏任务和微任务
- 宏任务（macrotask）：script（全局任务），setTimeout ，setInterval ，setImmediate **（node.js 独有）**，I/O（磁盘读写或网络通信） ，UI rendering（UI交互事件）
- 微任务（microtask）：process.nextTick **（node.js 独有）**, Promise.then, Object.observer（已废弃）, MutationObserver.

## 事件循环（event loop）
这里首先要明确一点：浏览器是一个进程，其有多个线程（具体见 Broswer 章节）
一般情况下, 浏览器有如下五个线程:
*   GUI 渲染线程
*   JavaScript 引擎线程
*   浏览器事件触发线程
*   定时器触发线程
*   异步 HTTP 请求线程


GUI 渲染线程和 JavaScript 引擎线程是互斥的，其他线程相互之间都是可以并行执行的。

<img src="https://box.kancloud.cn/6ada796292f275f364445670068e36af_730x760.png />


浏览器中，JavaScript 引擎循环地从任务队列中读取任务并执行，这种运行机制就叫做事件循环。
更准确地说，事件循环是按下面几个步骤执行的：
1.  执行同步代码，这属于宏任务（初始时的同步代码就是 script 整体代码）
2.  执行栈为空，查询是否有微任务 (microtask) 需要执行
3.  执行所有微任务
4.  必要的话渲染 UI
5.  然后开始下一轮 Event loop，执行宏任务中的异步代码

## 例题
```js
    setTimeout(function () {
      console.log(1);
    }, 0);
    
    Promise.resolve(function () {
      console.log(2);
    })
    
    new Promise(function (resolve) {
      console.log(3);
    });
    
    console.log(4);
    
    //上述代码的输出结果是什么？？？
```
正确答案是`------------------->` 3 4 1 
解释如下：
``` js
    // 遇到 setTimeout 将 setTimeout 回调放入宏仁务队列中
    setTimeout(function () {
      console.log(1)
    }, 0)

    // 遇到了 promise，但是并没有 then 方法回调 
    // 所以这句代码会在执行过程中进入我们当前的执行上下文 紧接着就出栈了
    Promise.resolve(function () {
      console.log(2)
    })

    // 遇到了一个 new Promise，Promise 有一个原则就是在初始化 Promise 的时候Promise 内部的构造器函数会立即执行，
    // 因此在这里会立即输出一个 3，所以这个 3 是第一个输出的
    new Promise(function (resolve) {
      console.log(3)
    })
    // 然后第二个输出 4  当代码执行完毕后回去微任务队列查找有没有任务，
    // 发现微任务队列是空的，那么就去宏仁务队列中查找，发现有一个我们刚刚放进去的setTimeout 回调函数，
    // 那么就取出这个任务进行执行，所以紧接着输出1
    console.log(4)
```

太简单了？来看看这题：
```js
console.log('begin'); // 1.begin
setTimeout(() => {
    console.log('setTimeout 1'); // 3.setTimeout 1
    Promise.resolve() // Promise.resolve().then ：直接把 then 回调放入微任务队列
        .then(() => {
            console.log('promise 1'); // 5.promise 1
            setTimeout(() => {
                console.log('setTimeout2'); // 8. setTimeout2
            });
        })
        .then(() => {
            console.log('promise 2'); // 7. promise 2  注意7比8要快，then 方法返回一个新的 Promise 对象
        });
    new Promise(resolve => {
        console.log('a'); // 4. a
        resolve();
    }).then(() => {
        console.log('b'); // 6. b
    });
}, 0);
console.log('end'); // 2.end
```
