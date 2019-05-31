---
layout: post
title:  "javascript模块化"
date:   2019-06-01 11:18:23 +0700
categories: [javascript]
---
>作者：myprelude@github  
原文链接： https://myprelude.github.io 
转载请注明出处，保留原文链接和作者信息。


## 为什么我们需要模块化

随着项目迭代，项目依赖越来越多，一个页面上有很多逻辑代码，如下

{% highlight html %}
<head>
    ...
    <script src='a.js'>
    <script src='b.js'>
    <script src='c.js'>
    ...
</head>

{% endhighlight %}

假如c.js依赖b.js，不小心颠倒了c.js和b.js的位置就会导致页面报错，同时要维护很多js脚本对开发人员来说也是一项很辛苦的任务。为了拯救开发于水火之中，社区也为此做来很大的努力。

## 努力一：闭包
由于太多的js文件导致维护依赖关系比较麻烦，如果全部写到一个js文件中呢？

{% highlight javascript %}
// a.js
(function(window,undefined){

    // 业务逻辑代码

})(window,undefined)

// b.js
(function(window,undefined){
    // 业务逻辑代码
})(window,undefined)
{% endhighlight %}
这样做减少变量冲突的问题，没有闭包里面的代码都是一个模块；但是并没有解决模块依赖问题。

## 努力二：AMD / CMD

### AMD
{% highlight javascript %}
// 定义一个模块 module
define(['Math','jQuery'],function(M,$){
    // do something
})

// 加载模块
require(['module'],function(module){
    // do something
})
{% endhighlight %}
### CMD
{% highlight javascript %}
// 定义一个模块
define(function(require,exports,module){
    // require 引入依赖
    // exports 导出一个模块
    // module 储存当前模块上的方法和变量
})

//  加载一个模块
seaJs.use(['module'],function(module){
    // do something
    var $ = require('jQuery');
    // do something
})
{% endhighlight %}
可以看出来AMD和CMD，都解决多js文件出现依赖错乱的关系。

* AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块 。
* CMD推崇就近依赖，只有在用到某个模块的时候再去require 。

AMD和CMD最大区别就是就是依赖处理机制不同。很多人都认为AMD代码是异步加载，CMD是同步加载；其实并不是的，AMD和CMD都是异步加载的，而AMD的依赖是前置的当一个模块加载完后就会执行该模块，只有当所有模块都加载完成才会执行require后面的回调函数；AMD将模块字符串解析一遍才知道需要加载那些模块，模块加载后并不是立即执行，只有单遇到require后才执行。

## 努力三：ES6模块化

{% highlight javascript %}
// 定义一个模块  module.js
export default const module = {
    name:'module',
    context:'this is es6 module'
}

// 加载一个模块
import module from  './module.js'
console.log(module.name)  // module
{% endhighlight %}

## 服务端的js模块化 CommonJS
CommonJS是node端模块化方案；解决nodejs 代码模块划分。
{% highlight javascript %}
// 定义一个模块  module.js
module.exports const module = {
    name:'module',
    context:'this is es6 module'
}

// 加载一个模块
const module = require('./module.js')
console.log(module.name)  // module
{% endhighlight %}

###  谈谈es6模块化和CommonJS的区别

* CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
* CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

### es6 模块化的优势

* es6 模块是代码静态分析，在编译阶段执行；开发者工具可以检测到一些 bug，通过webpack可以进行代码优化 如：Tree Shaking  Scope Hoisting。

## 模块循环加载问题

虽然模块循环加载在规范上是不容许的，但是随着项目体积增大难免会出现模块之间相互依赖，从而产生了依赖循环的场景，那么在js模块化是如何解决模块的循环加载呢？

### commonJS模块循环加载

commonJs在require模块时就执行了该模块，将require后的模块保存到内存中去，下次加载到该模块直接从内存中拿；

{% highlight javascript %}
// a.js
exports.done = true;
const b = require(./b.js)
console.log('在a中执行b模块b.done=${b.done}')
ecports.done = false;
console.log(`模块a执行完成`)
{% endhighlight %}
{% highlight javascript %}
// b.js
exports.done = true;
const a = require(./a.js)
console.log('在b中执行a模块a.done=${a.done}')
ecports.done = false;
console.log(`模块b执行完成`)
{% endhighlight %}
b.js执行到第二行，就会去加载a.js，这时，就发生了"循环加载"。系统会去a.js模块对应对象的exports属性取值，可是因为a.js还没有执行完，从exports属性只能取回已经执行的部分，而不是最后的值。

a.js已经执行的部分，只有一行。
{% highlight javascript %}
exports.done = true;
{% endhighlight %}
因此，对于b.js来说，它从a.js只输入一个变量done，值为true。

然后，b.js接着往下执行，等到全部执行完毕，再把执行权交还给a.js。于是，a.js接着往下执行，直到执行完毕。我们写一个脚本main.js，验证这个过程。

{% highlight javascript %}
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
{% endhighlight %}
执行main.js，运行结果如下。
{% highlight javascript %}
$ node main.js

在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true
{% endhighlight %}
上面的代码证明了两件事。一是，在b.js之中，a.js没有执行完毕，只执行了第一行。二是，main.js执行到第二行时，不会再次执行b.js，而是输出缓存的b.js的执行结果，即它的第四行。

### ES6模块循环加载

ES6模块的运行机制与CommonJS不一样，它遇到模块加载命令import时，不会去执行模块，而是只生成一个引用。等到真的需要用到时，再到模块里面去取值。
ES6根本不会关心是否发生了"循环加载"，只是生成一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。


