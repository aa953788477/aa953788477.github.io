---
title: 移动端页面多环境运行解决方案
date: 2017-04-17 14:30
tags:
- 移动web
---

在移动端的页面开发中，一个页面往往会在多个地方运行，例如：嵌入在app内、微信、其它浏览器..... ,在这些地方运行，很多功能的实现会采用不同方式，例如分享功能，在app中可能会调用native（android/ios）提供的接口，微信端会调用微信的sdk，这是两种完全不同的实现方式，但是对外的调用方式和接收参数、返回参数一致。

这时需要一个中间层(runtime)，对外调用runtime中的方法，runtime中区分运行环境来调用不同的实现方式。

<!--more-->

## 接口设计

首先我们确定不同运行环境的runtime 层的设计

```javascript
interface void use (Object runtime) {}
```

使用是，调用 `use` 方法加载不同运行环境下的实例。

然后就是不同运行环境的实例了

```javascript
{
    name // 运行环境名称
    boolean isRuntime () {}  // 环境判断
    // 具体的方法
    API: {
        share (options) {}
    }
}
```

* 通过 `name` 属性来区分不同的运行环境, 方便后续的方法扩展，
* isRuntime 方法，返回 true / false 提供判断是否是此运行环境的方法
* 接下来的就是其它方法了，不同运行环境的，接受参数和返回参数需要保持一致

## 实现

首先是 `runtime.js` 提供给外部调用，通过 `use` 方法加载不同运行环境的实现方式。

```javascript
const runtimes = []; // 用来保存不同的运行环境的示例
const Runtime = {
    use (runtime) {
        if (!runtime.name) {
            console.warn('must be have name');
        }
        // 判读此名称的运行环境, 有则将api合并，没有则加入到数组中
        if (!existMerge(runtime.name, runtime.API)) runtimes.push(plugin);
        createAPI(runtime.API); // 创建对面提供的方法
        return this;
    }
}

function existMerge (name, API) {
    let result = false;
    runtimes.forEach((runtime) => {
        if (runtime.name === name) {
            result = true;
            runtime.API = {
                ...runtime.API,
                ...API
            };
            return;
        }
    });
    return result;
}

function createAPI (API) {
    for (let methodName in API) {
        if (!API.hasOwnProperty(methodName)) continue;
        Runtime[methodName] = (function (methodName) {
            return function () {
                return callAPI(arguments, methodName);
            }
        })(methodName);
    }
}

function callAPI (args, methodName) {
    for (var i = 0; i < runtimes.length; i++) {
        const runtime = runtimes[i];
        if (!runtime.isRuntime()) continue;
        const API = runtime.API;
        if (!API[methodName] && typeof API[method] === 'function') continue;
        return API[methodName].apply(null, args);
    }
    console.warn(`找不到此方法:`, _module, method);
}

export default Runtime;
```

然后是不同运行环境下的实例

```javascript
// hybrid.js 嵌入到app中的页面
export default {
    name: 'hybrid',
    isRuntime () {
        return // TODO 通过userAgent判断运行环境
    },
    API: {
        share () {
            // TODO 调用native(android/ios) 提供接口
        }
    }
}
```

```javascript
// wechat.js 微信运行环境
export default {
    name: 'wechat',
    isRuntime () {
        return // TODO 通过userAgent判断运行环境
    },
    API: {
        share () {
            // TODO 调用微信sdk分享功能
        }
    }
}
```

最后使用

```javascript
import Runtime from './runtime';
import WeChat from './wechat';
import Hybrid from './hybrid';

Runtime.use(WeChat);
Runtime.use(Hybrid);

Runtime.share(); // 调用分享功能
```

这样一来我们可以在不同的运行环境下，给出不同的实现，使一个页面可以在多处运行。

