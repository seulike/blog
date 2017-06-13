### 这里只讨论在linux下的实现。
node事件循环的核心还是epoll。这也是nginx处理网络请求的核心。
实现一个简单的web服务器一般也是用linux的epoll来实现。epoll是用来处理io事件的。是select和poll的增强版本。网络上对epoll的介绍很多。能通过代码来了解是直接易懂的方式。本人的web server中用c实现了一个简单的服务器。通过这个可以对epoll的使用有更直观的了解。
目前的网站开发方案中有OpenResty就是利用Nginx+Lua开发进行网站开发。个人认为node的实现类似的有点像Nginx+javascript。
对于io的操作及处理。目前有各种封装好的库。node中使用的是[libuv](https://github.com/libuv/libuv).
### node中的核心代码
[循环最底层的代码](https://github.com/nodejs/node/blob/master/deps/uv/src/unix/core.c#L339)
核心方法为uv_run;循环中会对io事件及定时器等进行相应的处理。顺序依次为：
uv__run_timers(loop);uv__run_idle(loop);uv__run_prepare(loop);uv__io_poll(loop, timeout);uv__run_check(loop).
