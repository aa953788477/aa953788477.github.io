---
date: 2016-08-21 00:54
title: ES6的二三事(二)-模块化
tags: 
- javascript 
- ES6
---

在之前的 javascript 中一直是没有模块系统的，前辈们为了解决这些问题，提出了各种规范, 最主要的有CommonJS和AMD两种。前者用于服务器，后者用于浏览器。而 ES6 中提供了简单的模块系统，完全可以取代现有的CommonJS和AMD规范，成为浏览器和服务器通用的模块解决方案。

<!--more-->

## 基本用法

es6 中新增了两个命令 `export` 和 `import` , `export` 命令用于规定模块的对外接口，`import` 命令用于输入其他模块提供的功能。

> 一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个 变量，就必须使用export关键字输出该变量。下面是一个JS文件，里面使用export命令输出变量。

```javascript
// math.js

export const add = function (a, b) {
    return a + b
}

export const subtract = function (a, b) {
    return a - b
}
```

> 使用export命令定义了模块的对外接口以后，其他JS文件就可以通过import命令加载这个模块（文件）。

```javascript
// main.js
import { add, subtract } from './test.js'

add(1, 2)
substract(3, 2) 
```

## export 详细用法

上面介绍了模块化最基础的用法，export 不止可以导出函数，还可以导出对象，类，字符串等等

```javascript
export const obj = {
    test1: ''
}

export const test = ''

export class Test {
    constructor() {
    }
}
```

export的写法，除了像上面这样，还有另外一种。

```javascript
let a = 1
let b = 2
let c = 3
export {
    a,
    b,
    c
}
```

上面代码在export命令后面，使用大括号指定所要输出的一组变量。它与前一种写法是等价的，但是应该优先考虑使用这种写法。因为这样就可以在脚本尾部，一眼看清楚输出了哪些变量。

通过 as 改变输出名称

```javascript
// test.js
let a = 1
let b = 2
let c = 3
export {
    a as test,
    b,
    c
}
```

```javascript
import { test, b, c} from './test.js' // 改变命名后只能写 as 后的命名
```

上面啊的写法中，`import` 中需要指定加载的变量名或函数名，否则无法加载。但是，用户肯定希望快速上手，未必愿意阅读文档，去了解模块有哪些属性和方法。

`export default` 指定默认输出, import 无需知道变量名就可以直接使用

```javascript
// test.js

export default function () {
    console.log('hello world')
}
```

```javascript
import say from './test.js' // 这里可以指定任意变量名

say() // hello world
```

有了export default命令，加载模块时就非常直观了，以一些常用的模块为例

```javascript
import $ from 'jQuery'   // 加载jQuery 库
import _ from 'lodash'   // 加载 lodash
import moment from 'moment' // 加载 moment
```

## import 详细用法

`import` 为加载模块的命令，基础使用方式和之前一样

```javascript
// main.js
import { add, subtract } from './test'

// 对于export default 导出的

import say from './test'
```

### 通过 as 命令修改导入的变量名

```javascript
import {add as sum, subtract} from './test'

sum (1, 2)
```
### 加载模块的全部

除了指定输出变量名或者 `export.default` 定义的导入， 还可以通过 `*` 号加载模块的全部.

```javascript
// math.js

export const add = function (a, b) {
    return a + b
}

export const subtract = function (a, b) {
    return a - b
}
```

```javascript
import * as math from './test.js'

math.add(1, 2)
math.subtract(1, 2)
```

## 开始使用 ES6

上面介绍了，es6 中模块的使用方式，但是现在es6的模块化，无论在浏览器端还是 node.js 上都没有得到很好的支持，所以需要，一些转码的工具。使我们可以用es6的方式来编码，最后输出es5的代码。

这里推荐一款基于 es6 模块化方式的打包神器 [rollup](http://rollupjs.org/)，它使用 `Tree-shaking` 的技术打包，基本可以做到零冗余的代码，而且配置简单，打包速度也够快。

### 安装 rollup

首先在 `package.json` 中加上 `rollup` 打包依赖的包

```json
{
    // ...
    "devDependencies": {
        "babel-core": "^6.13.0",
        "babel-preset-es2015-rollup": "^1.1.1",
        "rollup": "^0.34.3",
        "rollup-plugin-babel": "^2.6.1",
        "rollup-plugin-commonjs": "^3.3.1",
        "rollup-plugin-node-resolve": "^2.0.0",
        "rollup-plugin-uglify": "^1.0.1"
    }
}
```

### 编写打包程序

下面是我的打包程序
```javascript
// build.js
var rollup = require('rollup');
var babel = require('rollup-plugin-babel');         // babel 插件
var uglify = require('rollup-plugin-uglify');       // js 混淆压缩插件
var npm = require('rollup-plugin-node-resolve');    // 使用第三方包依赖
var commonjs = require('rollup-plugin-commonjs');   // CommonJS模块转换为ES6
rollup.rollup({
    entry: 'src/index.js', //入口文件
    plugins: [  // 插件配置
        npm({ jsnext: true, main: true }),
        commonjs(),
        uglify(),
        babel({
            exclude: 'node_modules/**',
            presets: [ "es2015-rollup" ]
        })
    ]
}).then(function(bundle) {
    // 打包之后生成一个 `budble` 把它写入文件即可
    bundle.write({
        // 转化格式 cjs 代表 commonJs, 还支持 iife, amd, umd, es6 ....
        format: 'cjs', 
        banner: 'si_log.js v0.1.1',     //文件顶部的广告
        dest: 'dist/si_log_common.js'
    });
});
```

### 在 `package.json` 中加上执行脚本


```json
{
    // ...
    "scripts": {
        // ...
        "build": "node build.js"
    },
}
```

然后，执行命令
```shell
npm run build
```

ok 到这里打包就都结束了。

