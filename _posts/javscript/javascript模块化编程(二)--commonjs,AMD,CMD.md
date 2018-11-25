---
date: 2016-03-21 23:40
title: javascript模块化编程(二)--commonjs,AMD,CMD
tags: javascript
---

前面的写法，再文件模块比较少的情况都不会有问题，但是再当模块比较多之后，window下还是会被挂载很多对象，而且当模块依赖过多的时候需要在'（）'，写很长的参数，于是乎，前端模块化，有了一些规范，让我们去定义模块。
<!--more-->
## AMD
AMD是“Asynchronous Module Definition”的缩写，意思就是"异步模块定义"，它采用异步加载模块的方法，使用require的方法先加载定义上数组中的依赖模块，全部完成后执行回调函数。
```
require(['module1'], function(module1){
    module1.doSomethings();//调用模块方法
});
```
采用异步加载，不会使浏览器卡死。也可以定义一个模块
```
define(moduleId?, [modules]?, factory);
```
其中

> moduleId 模块Id可以忽略，忽略的时候加载使用文件路径
> modules 依赖模块,可以输模块id或是文件路径,没有可以忽略
> factory 加载完依赖之后的回调方法，这里是模块定义的具体实现，利用return语句返回模块

```
//注意依赖模块会返回对象，传入回调函数中
define('render', ['jQuery'], function($){
    //do somethings
    return {};
});
```
关于amd规范的实现：[require.js](http://www.requirejs.cn/)
## cmd规范
CMD（Common Module Definition）公共模块定义规范，
在 CMD 规范中，一个模块就是一个文件。代码的书写格式如下：
```javascript
define(factory)
```
define 是一个全局函数，用来定义模块。
factory可以是一个JSON,字符串等等...
```javascript
define({aa : "aa"});
define('ddd');
```
factory 为函数时，表示是模块的构造方法。执行该构造方法，可以得到模块向外提供的接口。factory 方法在执行时，默认会传入三个参数：*require*、*exports* 和 *module*：
```javascript
define(function(require, exports, module) {
    var module1 = require('module1');
    // 正确写法
    module.exports = {
        foo: 'bar',
        doSomething: function() {}
    };
});
```

> require 用于加载模块依赖；
> exports 导出对象，等同于module.exports;
> module 模块对象，用于使用module.exports导出对象;

值得注意的是exports 仅仅是 module.exports 的一个引用。在 factory 内部给 exports 重新赋值时，并不会改变 module.exports 的值。因此给 exports 赋值是无效的，不能用来更改模块接口。
cmd规范实现：[sea.js](http://seajs.org/)
## amd和cmd规范对比；
相较于amd而言cmd的规范的优势很明显

> 1.按需加载, amd将依赖前置到模块定义时，也就是有些模块尚未使用到就加载了，暂用内存，不合理.
> 2.CMD推崇依赖就近，AMD推崇依赖前置。当依赖模块过多时，回调函数如会有很长一条参数对象.

## commonjs
CommonJs 是服务器端模块的规范，Node.js采用了这个规范。
根据CommonJS规范，一个单独的文件就是一个模块。加载模块使用require方法，该方法读取一个文件并执行，最后返回文件内部的exports对象。
```
var test="a";//私有变量
var $ = require('jQuery');//拉取文件依赖
exports.aa = function (){

}
exports.bb =  function(){

}
//也可以这么写
module.exports = {}
```
CommonJS 加载模块是同步的，所以只有加载完成才能执行后面的操作。像Node.js主要用于服务器的编程，加载的模块文件一般都已经存在本地硬盘，所以加载起来比较快，不用考虑异步加载的方式，所以CommonJS规范比较适用。但如果是浏览器环境，要从服务器加载模块，这是就必须采用异步模式。所以就有了 AMD  CMD 解决方案。
现在利用一些前端打包工具，我们也可以是使用标准的额common.js规范编码了，[browserify](http://browserify.org/),[webpack](http://webpack.github.io/);

## umd
我们编写一个模块，并不知道它要在哪种规范下使用，于是，需要一种编写方式能同事兼容Amd和CommonJs模块化实现，这就是umd
```
(function (window, factory) {
    if (typeof exports === 'object') {
        module.exports = factory();
    } else if (typeof define === 'function' && define.amd) {
        define(factory);
    } else {
        window.module1 = factory();
    }
})(this, function () {
    //module ...
});
```
UMD先判断是否支持Node.js的模块（exports）是否存在，存在则使用Node.js模块模式。
在判断是否支持AMD（define是否存在），存在则使用AMD方式加载模块。
如果都不存在模块化实现那么就放在window对象下。

## ES6模块化
历史上，JavaScript一直没有模块（module）体系，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。其他语言都有这项功能，比如Ruby的require、Python的import，甚至就连CSS都有@import，但是JavaScript任何这方面的支持都没有，这对开发大型的、复杂的项目形成了巨大障碍。

在ES6之前，社区制定了一些模块加载方案，最主要的有CommonJS和AMD两种。前者用于服务器，后者用于浏览器。ES6在语言规格的层面上，实现了模块功能，而且实现得相当简单，完全可以取代现有的CommonJS和AMD规范，成为浏览器和服务器通用的模块解决方案。

ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS和AMD模块，都只能在运行时确定这些东西。比如，CommonJS模块就是对象，输入时必须查找对象属性。
ES6模块不是对象，而是通过export命令显式指定输出的代码，输入时也采用静态命令的形式。
```javascript
// ES6模块
import { stat, exists, readFile } from 'fs';
```
上面代码的实质是从fs模块加载3个方法，其他方法不加载。这种加载称为“编译时加载”，即ES6可以在编译时就完成模块加载，效率要比CommonJS模块的加载方式高。当然，这也导致了没法引用ES6模块本身，因为它不是对象。
### export命令
模块功能主要由两个命令构成：export和import。export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。

一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用export关键字输出该变量。下面是一个JS文件，里面使用export命令输出变量。
```
export var name1 = 'Michael';
export var name2 = 'Jackson';
export var name3 = 1958;
```
### import命令
使用export命令定义了模块的对外接口以后，其他JS文件就可以通过import命令加载这个模块（文件）。
```
//
import ajax from '../ajax.js'
```
### export default命令
```
// export-default.js
export default function () {
  console.log('foo');
}
```
上面代码是一个模块文件export-default.js，它的默认输出是一个函数。
```
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```
其他模块加载该模块时，import命令可以为该匿名函数指定任意名字。
Es6中已经提供了比较完善的模块化方案，但是目前兼容不太好，只能通过babel转码成es5代码再通过webpack打包.
ES6模块化这部门讲的比较粗糙，如果感兴趣可以看下阮一峰大大的教程
[ECMAScript6入门-Module](http://es6.ruanyifeng.com/#docs/module)