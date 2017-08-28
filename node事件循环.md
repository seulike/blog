### 前言
这里只讨论在linux下的实现。node事件循环的核心还是epoll。这也是nginx处理网络请求的核心。
实现一个简单的web服务器一般也是用linux的epoll来实现。epoll是用来处理io事件的。是select和poll的增强版本。网络上对epoll的介绍很多。能通过代码来了解是直接易懂的方式。本人的项目web server中用c实现了一个简单的服务器。通过这个可以对epoll的使用有更直观的了解。
目前的网站开发方案中有OpenResty就是利用Nginx+Lua开发进行网站开发。个人认为node的实现类似的有点像Nginx+javascript。
对于io的操作及处理。目前有各种封装好的库。node中使用的是[libuv](https://github.com/libuv/libuv)。  
关于node,本人有个相关的[分享](https://github.com/seulike/blog/blob/master/doc/node%E5%8E%9F%E7%90%86%E4%BB%8B%E7%BB%8D.pdf)
### node中的核心代码
[循环最底层的代码](https://github.com/nodejs/node/blob/master/deps/uv/src/unix/core.c#L339) 
底层在libuv库中，核心方法为uv_run;循环中会对io事件及定时器等进行相应的处理。顺序依次为：
uv__run_timers(loop);uv__run_idle(loop);uv__run_prepare(loop);uv__io_poll(loop, timeout);uv__run_check(loop).
1. uv__run_timers最开始执行定时器里面的任务。即由setTimeout和setInterval添加的定时器事件。由setInterval的实现可以看到。其是通过到达间隔事件后再次调用setTimeout来实现。对于定时器事件的存储组织，很早的实现版本使用了红黑树。找寻节点中timeout小于当前时间time的定时器并执行回调函数。当前版本使
[最小堆](https://github.com/nodejs/node/blob/master/deps/uv/src/unix/timer.c#L150)
依次取出最小值，比较timeout是否小于当前时间，是就执行定时器的回调函数。直到堆中元素耗尽或节点的timeout未超时。
2. uv__io_poll则是对添加到epoll中的io事件进行回调处理。包含文件io及网络io等。
3. uv__run_check中则处理了process.setImmediate里面添加的事件。


