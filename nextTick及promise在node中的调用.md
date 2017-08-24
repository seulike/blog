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

## node中的事件循环
核心在于uv_run方法：顺序是：uv__run_timers(处理定时器事件，setTimeout，setInterval)。
uv__io_poll（处理添加到epoll中的io事件）。uv__run_check（setImmediate）。 
因此macrotasks，microtasks的概念应该只能适用于html的事件循环。至于node事件循环与html并不一致。node使用v8引擎来解析js，并遵循ECMA-265规范。在es6中microtasks被称为jobs。Promises则属于jobs。这里把setImmediate放入macrotasks，process.nextTick放入microtasks，是因为在node的实现中process.nextTick有点类似microtasks。

## node中process.nextTick及Promises的调用步骤
对于这一问题可看[知乎](https://www.zhihu.com/question/23028843)
同时这里会给出我的理解。 
process.nextTick的[实现代码](https://github.com/nodejs/node/blob/master/lib/internal/process/next_tick.js#L49)
process.nextTick是node中实现的api，而promise的相关操作由v8来实现。  
node在初始化运行时，会加载运行(/lib/internal/bootstrap_node.js)。
其中会加载运行next_tick.js。
```NativeModule.require('internal/process/next_tick').setup();```
在setup方法中:
```process.nextTick = nextTick;
process._tickCallback = _tickCallback;
const tickInfo = process._setupNextTick(_tickCallback, _runMicrotasks);
  _runMicrotasks = _runMicrotasks.runMicrotasks;
```
设置了process.nextTick，这个即是使用的api。这里维护了一个NextTickQueue，会将设置的回调函数加入到队列中。
由env->SetMethod(process, "_setupNextTick", SetupNextTick);
可知这里会调用[SetupNextTick](https://github.com/nodejs/node/blob/master/src/node.cc#L1241)。
```
env->set_tick_callback_function(args[0].As<Function>());
env->SetMethod(args[1].As<Object>(), "runMicrotasks", RunMicrotasks);
```
这里设置了tick_callback_function为_tickCallback，_runMicrotasks.runMicrotasks = RunMicrotasks;RunMicrotasks最终会调用v8的api,RunMicrotasks,用于执行所有的微任务。

在node的事件循环中函数uv__io_poll是对添加到epoll中的io事件进行回调处理。
这里的处理就如同用户代理处理macrotask一样。这些事件的最终执行[回调函数]
(https://github.com/nodejs/node/blob/master/src/async-wrap.cc#L661)
在执行完用户设置的回调函数后，会调用tick_callback_function。即已经在next_tick.js中设置好的函数[_tickCallback](https://github.com/nodejs/node/blob/master/lib/internal/process/next_tick.js#L151)。 
从nextTickQueue中不停取出数据并执行其callback，取出个数限制为1000。
之后_runMicrotasks，执行所有的微任务。 





