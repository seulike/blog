## 前言
使用饿了么前端的vue组件vue-infinite-scroll。
调用代码如下:
<div v-infinite-scroll="loadMore()"loadMore()
     infinite-scroll-disabled="busy"
     infinite-scroll-distance="20">
	<li v-for="item in listData"><span>{{ item }}</span></li>
</div>
发现会无限加载。这个问题也被提了[issue](https://github.com/ElemeFE/vue-infinite-scroll/issues/35);
解决方法很简单loadMore()改成loadMore。本文探讨产生这个问题的原因。

## 找寻原因
在代码中搜索loadMore，可以找到相关代码。
在这里_vm.loadMore(),调用了loadMore方法。根据断点调试，这里会进入编写的loadMore方法并加载数据。
然后继续调试。根据调试过程的函数调用，可以大致可以理出原因。loadMore加载数据，导致数据发生变化，因而又导致dom更新。dom更新会调用这里的render方法。
导致调用loadMore加载数据，如此循环致使一直加载数据。

## 进一步了解
  render函数的生成。我在调试时没有发现render函数的生成过程。而是在源码中直接有。原来是由vue-loader生成的。
  webpack的配置文件中对于.vue后缀的文件会使用vue-loader来加载。vue-loader会将template转换成render函数。
  查看源码在/lib/parse.js中,会使用vue-template-compiler模块来解析模板内容。
  ```
  var compiler = require('vue-template-compiler')
  output = compiler.parseComponent(content, { pad: 'line' })
  ```
  对template解析会得到render函数。得到的代码如下：
  ![image](https://github.com/seulike/blog/blob/master/img/loadMore1.png)
  其中查看vue-template-compiler源码。genDirectives方法会对指令进行相应处理。
  ```
  res += "{name:\"" + (dir.name) + "\",rawName:\"" + (dir.rawName) + "\"" + (dir.value ? (",value:(" + (dir.value) + "),expression:" + (JSON.stringify(dir.value))) : '') + ...
  ```
  其中由于属性value直接使用了loadMore(),而属性expression则使用了字符串"loadMore()"。因而导致调用render方法是会调用loadMore()。
  vue组件的生命周期在官网上有说明。对组件的初始化在方法Vue.prototype._init。里面组件会经历beforeCreate,Observe Data,Init Events,created等阶段。
  最后会调用mountComponent方法。
  ```
  updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  vm._watcher = new Watcher(vm, updateComponent, noop)
  ```
  这里会设置updateComponent方法。并添加新的watcher。这样一旦数据进行了更新就会调用updateComponent。而updateComponent会调用到render函数，
  而render函数里面会调用loadMore()，导致数据更新。如此循环。
  
  
  
  
  
  



