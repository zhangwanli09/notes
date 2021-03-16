# js运行机制（Event Loop）

### 单线程

JavaScript是单线程的，也就是同一时间只能做一件事。这是js的核心特征。单线程意味着所有任务需要`排队`，前一个任务结束才会执行下一个任务。

为什么不是多线程？js的主要用途是处理交互操作DOM，这决定了它只能是单线程，否则会带来复杂的同步问题。（假定js同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？）。

还有人说js还有Worker线程，对的，为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程是完全受主线程控制的，而且不得操作DOM。所以，这个标准并没有改变JavaScript是单线程的本质。

### 任务队列

将任务分成两种：
1. `同步任务`：在主线程排队执行的任务，前一个任务执行完毕，才能执行后一个任务。
2. `异步任务`：不进入主线程，而进入`任务队列`的任务，只有任务队列通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

`事件循环`大致有以下几个步骤：
1. 所有同步任务都在主线程上执行，形成一个`执行栈`。
2. 主线程之外，存在一个`任务队列`，当异步任务`有结果`，就会在任务队列中放置一个`事件`。
3. 一旦`执行栈`中所有同步任务`执行完毕`，系统就会读取`任务队列`，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
4. 主线程不断`重复`上面的第3步。

![主线程和任务队列](../imgs/img4.png ':size=500')

只要主线程空了，就会去读取`任务队列`，这个过程会不断重复。这就是JavaScript的运行机制。

任务队列（事件队列），每完成一个异步任务就会在任务队列中添加一个事件，表示相关的异步任务可以进入执行栈。主线程读取任务队列，就是读取里面有哪些事件。任务队列中的事件也包括一些用户产生的事件（点击事件，页面滚动事件等）。只要指定过回调函数，这些事件发生时就会进入任务队列，等待主线程读取。

回调函数就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

任务队列是一个先进先出的数据结构，排在前面的事件优先被主线程读取。主线程的读取基本上是自动的，只要执行栈一清空，任务队列上第一位事件就会自动进入主线程。由于可能存在定时器（setTimeout，setInterval）功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。

### Event Loop

js分为`同步任务`和`异步任务`。

同步任务都在主线程（js引擎线程）上执行，会形成一个`执行栈`。

主线程之外，事件触发线程管理着一个`任务队列`，只要异步任务有了运行结果，就在任务队列里放一个事件回调。

一旦`执行栈`中所有同步任务执行完毕（也就是js引擎线程空闲了），系统就会读取`任务队列`，将可运行的异步任务添加到执行栈中执行。（只要任务队列中有事件回调，就说明可以执行）。

主线程（js引擎线程）从任务队列中读取事件的过程是循环不断的，这种运行机制称为`Event Loop`（事件循环）。

执行栈中的代码（同步任务），总是在任务队列（异步任务）之前执行。

![Event Loop](../imgs/img3.png ':size=500')

### 定时器（timer）

任务队列除了可以放置异步任务的事件，还可以放置定时事件（`setTimeout`，`setInterval`）。

如果将`setTimeout`的第二个参数设为`0`，表示当前代码`执行栈清空`后，立即执行（0毫秒间隔）指定的回调函数。

```javascript
console.log(1)
setTimeout(() => { console.log(2) }, 0)
console.log(3)

// 打印 1 3 2
```

`setTimeout(fn, 0)`的含义是指定某个任务在主线程最早可得的空闲时间执行（尽可能早的执行）。它在任务队列尾部添加一个事件，要等同步任务和任务队列现有的事件都处理完再执行，所有不能保证一定会在参数设置的时间执行。

### 宏任务（macro task）、微任务（micro task）

规范中规定`task`分为两大类，分别是`macro task`和`micro task`。

#### 宏任务

将每次执行栈执行的代码当作一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行）。每个宏任务会从头到尾执行完毕，不会执行其他。

由于`JS引擎线程`和`GUI渲染线程`是互斥的关系，浏览器为了能够使宏任务和DOM任务有序的进行，会在一个宏任务执行结束后，在下一个宏任务执行前，GUI渲染线程开始工作，对页面进行渲染。

```javascript
// 宏任务 -> GUI渲染 -> 宏任务 -> ...
```

常见的宏任务：
1. 主代码块
2. setTimeout
3. setInterval
4. MessageChannel
5. postMessage
6. setImmediate()（Node）
7. requestAnimationFrame()（浏览器）

#### 微任务

微任务可以理解成在当前宏任务执行后立即执行的任务。当一个宏任务执行完，会在渲染前，把执行期间产生的所有微任务都执行完。

```javascript
// 宏任务 -> 微任务 -> GUI渲染 -> 宏任务 -> ...
```

常见的微任务：
1. process.nextTick()（node）
2. Promise.then()
3. catch
4. finally
5. Object.observe
6. MutationObserver

#### 注意点

1. 浏览器会先执行一个宏任务，接着执行当前执行栈产生的微任务，再进行渲染，然后再执行下一个宏任务。
2. 微任务和宏任务不在一个任务队列。
    1. 例如setTimeout是一个宏任务，它的事件回调在宏任务队列，Promise.then()是一个微任务，它的事件回调在微任务队列，二者并不是一个任务队列。
    2. 以Chrome为例，有关渲染的都是在渲染进程中执行，需要主线程执行的任务都会在主线程中执行，而浏览器维护了一套事件循环机制，主线程上的任务都会放到消息队列中执行，主线程会循环消息队列，并从头部取出任务进行执行，如果执行过程中产生其他任务需要主线程执行的，渲染进程中的其他线程会把该任务塞入到消息队列的尾部，消息队列中的任务都是宏任务。
    3. 微任务是如何产生的？当执行到script脚本的时候，js引擎会为全局创建一个执行上下文，在该执行上下文中维护了一个微任务队列，当遇到微任务，就会把微任务回调放在微队列中，当所有的js代码执行完毕，在退出全局上下文之前引擎会去检查该队列，有回调就执行，没有就退出执行上下文，这也就是为什么微任务要早于宏任务，每个宏任务都有一个微任务队列（由于定时器是浏览器的API，所以定时器是宏任务，在js中遇到定时器也会放到浏览器的队列中）。

#### 流程图

![宏任务和微任务](../imgs/img5.png ':size=400')

### 完整 Event Loop 流程图

![Event Loop](../imgs/img7.png ':size=700')

1. 首先，整体的script（作为第一个宏任务）开始执行时，把所有代码分为`同步任务`和`异步任务`两部分。
2. 同步任务会直接进入主线程依次执行。
3. 异步任务会再分为宏任务和微任务。
4. 宏任务进入到Event Table中，并在里面注册回调函数，每当指定的事件完成时，Event Table会将这个函数移到Event Queue中。
5. 微任务也会进入到另一个Event Table中，并在里面注册回调函数，每当指定的事件完成时，Event Table会将这个函数移到Event Queue中。
6. 当主线程内的任务执行完毕，主线程为空时，会检查微任务的Event Queue，如果有任务，就全部执行，如果没有就执行下一个宏任务。
7. 上述过程会不断重复，这就是Event Loop，比较完整的事件循环。

### 关于Promise

```javascript
console.log(1)

new Promise((resolve, reject) => {
  console.log(2)
  setTimeout(function () {
    console.log(3)
    resolve('成功')
  }, 0)
  console.log(4)
}).then(res => {
  console.log(5)
})

console.log(6)

// 打印结果 1 2 4 6 3 5
```

前面`new Promise`部分是同步任务，后面`.then`才是一个异步微任务。

### 关于async/await

```javascript
setTimeout(() => { console.log(4) }, 0)

async function fn () {
  console.log(1)
  await Promise.resolve()
  console.log(3)
}

fn()

console.log(2)

// 打印结果 1 2 3 4
```

可以理解为`await`之前的代码相当于`new Promise`的同步代码，`await`之后的代码相当于`Promise.then`的异步。

### 练手题

```javascript
function test() {
  console.log(1)
  // timer1
  setTimeout(function () {
    console.log(2)
  }, 1000)
}

test()

// timer2
setTimeout(function () {
  console.log(3)
}, 0)

new Promise(function (resolve) {
  console.log(4)
  // timer3
  setTimeout(function () {
    console.log(5)
  }, 100)
  resolve()
}).then(function () {
  // timer4
  setTimeout(function () {
    console.log(6)
  }, 0)
  console.log(7)
})

console.log(8)

// 打印 1 4 8 7 3 6 5 2
```

1. 执行test()，test方法为同步任务，直接执行console.log(1)打印`1`。
2. test方法中setTimeout为异步宏任务，回调记做timer1放入宏任务队列。
3. 接着执行，test方法下面的setTimeout为异步宏任务，回调记做timer2放入宏任务队列.
4. 接着执行，new Promise是同步任务，直接执行，打印`4`.
5. new Promise里面的setTimeout是异步宏任务，回调记做timer3放入宏任务队列.
6. Promise.then是微任务，放到微任务队列。
7. console.log(8)是同步任务，直接执行，打印`8`。
8. 主线程任务执行完毕，检查微任务队列中有Promise.then。
9. 开始执行微任务，发现有setTimeout是异步宏任务，回调记做timer4放入宏任务队列。
10. 微任务队列中的console.log(7)是同步任务，直接执行，打印`7`。
11. 微任务执行完毕，第一次循环结束。
12. 检查宏任务队列，里面有timer1、timer2、timer3、timer4，四个定时器宏任务，按照定时器延迟时间得到执行顺序，Event Queue：timer2、timer4、timer3、timer1，依次拿出放入执行栈末尾执行（浏览器Event Loop的Macrotask queue，就是宏任务队列在每次循环中只会读取一个任务）。
13. 执行timer2，console.log(3)为同步任务，直接执行，打印`3`。
14. 检查没有微任务，第二次Event Loop结束。
15. 执行timer4，console.log(6)为同步任务，直接执行，打印`6`。
16. 检查没有微任务，第三次Event Loop结束。
17. 执行timer3，console.log(5)为同步任务，直接执行，打印`5`。
18. 检查没有微任务，第四次Event Loop结束。
19. 执行timer1，console.log(2)为同步任务，直接执行，打印`2`。
20. 检查没有微任务，也没有宏任务，第五次Event Loop结束。

最终打印结果：1 4 8 7 3 6 5 2


### Node.js中的运行机制

虽然NodeJS中的JavaScript运行环境也是V8，也是单线程，但还是有些区别。

nodejs的宏任务分好几种类型，而这好几种又有不同的任务队列，而不同的任务队列又有顺序区别，而微任务是穿插在每一种宏任务之间的。

在node环境下，`process.nextTick`的优先级高于Promise，可以简单理解为在宏任务结束后会先执行微任务队列中的nextTickQueue部分，然后才会执行微任务中的Promise部分。

> 参考资料：
>
> https://juejin.cn/post/6844904050543034376#heading-22
>
> https://www.ruanyifeng.com/blog/2014/10/event-loop.html
