---
title: 简化 vuex 的状态管理方案
date: 2018-6-1 17:34
tags: 
- vue
---

在 vuejs 相关项目开发过程中，我们常常会使用 `vuex` 作为状态管理工具, 整个组件的状态做为单向数据流的模式管理。

![](https://vuex.vuejs.org/vuex.png)

事实上，在实际的使用中 vuex 可以说是相当繁琐的，每一次的需求增加需要增加 `Mutations-Type` 、`Action` 和 `Mutations`, 为了简化这一操作，我们可以将 mutations 和 action 合并，简化流程如下：

![](https://vuex.vuejs.org/flow.png)

在此种思想的引导下, [muse-model](https://github.com/myronliu347/muse-model) 诞生了，以简单优雅的方式完成整个项目的状态管理。
<!--more-->
## 什么是 muse-model

`muse-model` 并不是一个全新的状态管理工具， 它是基于 `vuex` 开发，可以说是 `vuex` 的一个辅助工具，在使用 `muse-model` 过程中，vuex 的一切 API 都是可以用的，这也方便了vuex 的用户进行过度。在初始化 `muse-model` 是也是需要传入 `store` 对象。

```javascript
// model.js
import Vue from 'vue';
import Vuex from 'vuex';
import MuseModel from 'muse-model';

export const store = Vuex.Store({
  strict: true
});

export default new MuseModel(store);
```

## 使用

我们将以一个计数器的例子来演示 `muse-model` 的使用。

### 定义一个 model

model 由 `namespace`、`state`、`action` 三个部分组成

```javascript
// count.js
export default {
    namespace: 'demo',
    state: {
        count: 1
    },
    add () {
        return {
            count: this.state.count + 1
        }
    },
    sub () {
        return {
            count: this.state.count - 1
        }
    }
}
```

不要再 `action` 中直接改变状态，而是通过 `return` 返回需要改变的新的状态.

### 连接组件

通过 `connect` 方法可以将 model 混入到组件的 `computed` 和 `methods` 中。

```html
<template>
<div>
    <button @click="add">+</button>
    {{count}}
    <button @click="sub">-</button>
</div>
</template>
<script>
import model from './model';
import CountModel from './count';
const CountUI = {
    name: 'count-ui'
};

export default model.connect(CountUI, CountModel);
</script>
```

## 处理异步

关于异步处理只需要返回 promise 对象即可。

```javascript
export default {
    //....
    addTimeOut () { // 异步处理
        return new Promise((resolve, reject) => {
          setTimeout(() => {
            resolve({
              count: this.state.count + 1
            });
          }, 1000);
        });
    }
}
``` 

## 资源

muse-model 地址：https://github.com/myronliu347/muse-model


