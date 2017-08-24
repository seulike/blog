## 前言
为了整合事件，用户交互，脚本，渲染，网络等。用户代理（浏览器）一般会使用事件循环（event loop）。[html标准](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)
定了事件循环及相应操作。用户代理会将所有的来自同一任务源的任务添加到同一个任务队列（task queue）。事件循环中会有一个task queues及一个微任务队列（microtask queue）。
一般将task queues中的任务称为macrotasks。 一般的macrotasks和microtasks[分类如下](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)： 
1. macrotasks: setTimeout, setInterval, setImmediate, requestAnimationFrame, I/O, UI rendering
2. microtasks: process.nextTick, Promises, Object.observe, MutationObserver

## 事件循环运行步骤
1. 从task queue从获取一个任务并执行，然后从队列中将其移除。
2. 执行微任务检查：从微任务队列中获取一个任务并执行直至队列为空。
3. 更新rendering。

## node中process.nextTick及Promises的调用步骤
对于这一问题可看[知乎](https://www.zhihu.com/question/23028843)
process.nextTick的[实现代码](https://github.com/nodejs/node/blob/master/lib/internal/process/next_tick.js#L49)
process.nextTick是node中实现的api，而promise的相关操作由v8来实现。
在node的是事件循环中




