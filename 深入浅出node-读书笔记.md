# 深入浅出node-读书笔记  

### 一.模块机制
##### CommonJS 模块规范

![](/images/1528425635mw.png)

1.模块中 module 对象代表模块自身 

2.exports 对象是module 的属性， 用于导出当前模块的方法或变量

3.node引入模块，3个步骤  
（1）路径分析  
（2）文件定位  
（3）编译执行   

4.模块分类：  
node自身的 核心模块，源代码编译中，编译进了二进制执行文件；加载进内存，速度最快.  
用户编写的文件模块，运行时动态加载，需要进行完整的路径分析，文件定位，编译执行. 

5.require 引入的加载过程  
（1）首先从缓存加载：  
无论核心模块还是文件模块，require 方法对相同模块的二次加载都一律采用缓存优先的方式。  
node 缓存的是 编译执行之后的对象。

> tips：标识符分类  
a. 核心模块，如 http，fs, path等  
b. 以 . 或 .. 开始的相对路径文件  
c. 以 / 开始的绝对路径文件  
d. 非路径形式的文件模块，如自定义模块 XX

（2）核心模块的加载：仅次于缓存  
（3）路径形式的文件模块加载：  
将 相对路径，绝对路径，转换成真实路径加载，并将真实路径作为索引将编译执行后结果缓存。  
（4）自定义模块加载：  
> tips  
模块路径：node 在文件模块的具体文件时制定的查找策略，具体为路径组成的数组  
操作：  
a. 创建文件，内容 console.log(module.paths)  
b. 执行   
可以看到，沿当前路径向上逐级递归，直到根目录下的 node_modules目录，可见当前文件路径越深，查找越耗时。

文件查找：缺少扩展名，按照 .js, .json, .node 的顺序依次尝试  
目录分析和包：当查找到的标识是一个目录，node 会将其当做包来处理  
包定位过程：   
   - 查找 package.json ，通过 JSON.parse() 解析出包描述对象，取出 main 属性指定的文件名进行定位 ；
   - 如果缺少扩展名， 进入扩展名分析步骤；
   - 如果 main 属性没指定或指定错误，node 将index 当做默认文件名，依次查找 index.js, index.jaon, index.node; 
   - 没有定位成功任何文件，则进入路径数组下一个路径进行查找。
   
6.模块编译  
- .js: 通过 fs 模块同步读取文件后执行。  
- .node:  c/c++  
- .json:  JSON.parse 解析  
- 其余扩展名: 当做 .js 文件载入

7.npm 常用功能  
全局模式安装（如果包中含有命令行工具），实际上 -g 是将一个包安装为全局可用的可执行命令。根据描述文件中的bin 字段配置，将实际脚本链接到与 node 可执行文件相同的路径下。  
如果node可执行文件的位置是 /usr/local/bin/node ，那么模块路径就是 /usr/local/bin/node_modules。最后，通过软链接方式将bin字段配置的可执行文件链接到 node 可执行目录下。  

8.AMD & CMD

9.ES6 模块 与 CommonJS 规范区别
> http://es6.ruanyifeng.com/#docs/module-loader  

>   - CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。 
>   - CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
   
   Node 规定 ES6 模块之中不能使用 CommonJS 模块的特有的一些内部变量。  
   首先，就是this关键字。ES6 模块之中，顶层的this指向undefined；CommonJS 模块的顶层this指向当前模块，这是两者的一个重大差异。  
   其次，以下这些顶层变量在 ES6 模块之中都是不存在的。

      arguments
      require
      module
      exports
      __filename
      __dirname

- 循环引用  
（1）一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。  
（2）需要开发者自己保证，真正取值的时候能够取到值。

> [官方文档](https://nodejs.org/api/modules.html#modules_cycles)

10.兼容多种模块规范  
```javascript
(function (name, definition) {
	var hasDefine = typeof define === 'function';
	var hasExports = typeof module !== 'undefined' && module.exports;

	if (hasDefine) {
		define(definition);
	} else if (hasExports) {
		module.exports = definition();
	} else {
		this[name] = definition();
	}
})('hello', function () {
	var hello = function () {};
	return hello;
})
```

### 二.异步IO  
1.为什么要异步IO

- 用户体验  
浏览器 javascript 在单线程上执行，并且与UI渲染共用一个线程，同步情况下，当脚本执行时，UI渲染阻塞。  
- 资源分配  
单线程阻塞IO使硬件资源得不到最优使用；多线程存在死锁、状态同步等问题！  
Node 提供类似Web Workers 的子进程。  

2.Node 的异步IO  
（1）异步IO的几个关键词：单线程、事件循环、观察者 和 IO线程池  
（2）除了用户代码不能并行执行以外，所有IO都是可以并行起来。  

3.其他异步 API  
setTimeout(),setInterval(),process.nextTick()  
(1) setTimeout(),setInterval() 创建的定时器会被插入到定时器观察者内部的一个红黑树中；process.nextTick() 只会将回调函数放入队列中。前者复杂度O(lg(n)),后者O(1),后者更高效。

