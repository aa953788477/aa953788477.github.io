---
title: Muse UI — Vue2.0 和 Material Design 结合
date: 2016-11-12 08:34
tags: 
- vue
---

Vue 2.0 发布以来，很多 vue 的开源项目都开始了升级计划，我也思考着 `vue-carbon` 的升级之路，9月开工，11月完工， [Muse UI](https://museui.github.io/) 闪亮登场。

<!--more-->

## 先睹为快

Muse UI 主要用于移动端和一些对浏览器兼容性要求不高的桌面端应用，先上地址:

[https://github.com/museui/muse-ui](https://github.com/museui/muse-ui)

官网和文档在这：

[https://museui.github.io/](https://museui.github.io/)

## 特性

* 基于 vue2.0 开发

* 组件丰富

* 丰富的主题，支持自定义主题

* 可以很好的配合 vue 的其它插件vue-router , vue-validator 使用

* 友好的 API

## 使用


```bash
npm install muse-ui --save
```

### 完整引入

```javascript
import Vue from 'vue'
import MuseUI from 'muse-ui'
import 'muse-ui/dist/muse-ui.css'
Vue.use(MuseUI)
```

### 按需引入

首先需要需要修改 `webpack` 的配置

```javascript
{
  // ...
  module: {
    loaders: [
      {
        test: /muse-ui.src.*?js$/,
        loader: 'babel'
      }
    ]
  },
  resolve: {
    // ...
    alias: {
      'muse-components': 'muse-ui/src'
    }
  }
}
```

`main.js`

```javascript
import Vue from 'vue'
import 'muse-components/style/base.less' // 全局样式包含 normalize.css
import appbar from 'muse-components/appbar'
import avatar from 'muse-components/avatar'
import {bottomNav, bottomNavItem} from 'muse-components/bottomNav'
import {retina} from 'muse-components/utils'

retina() // 1px 处理方案

// ...
Vue.component(appbar.name, appbar)
Vue.component(avatar.name, avatar)
Vue.component(bottomNav.name, bottomNav)
Vue.component(bottomNavItem.name, bottomNavItem)
```

### 示例 bottomNav 的使用

```html
<template>
 <mu-bottom-nav :value="bottomNav" shift @change="handleChange">
    <mu-bottom-nav-item value="movies" title="Movies" icon="ondemand_video"/>
    <mu-bottom-nav-item value="music" title="Music" icon="music_note"/>
    <mu-bottom-nav-item value="books" title="Books" icon="books"/>
    <mu-bottom-nav-item value="pictures" title="Pictures" icon="photo"/>
</mu-bottom-nav>
</template>
<script>
export default {
  data () {
    return {
      bottomNav: 'movies'
    }
  },
  methods: {
    handleChange (val) {
      this.bottomNav = val
    }
  }
}
</script>
```

![bottomNav](/images/bottomNav.gif)

## 关于 Muse

为了配合Vue 2.0 改变了 `vue-carbon` 许多的 API，新增了许多的组件，由于改变的太多，于是更名为 Muse UI，做为一个全新的 UI 框架。

`Muse` 取自于古希腊神话中的女神，掌管科学与艺术。我希望 `Muse` 和 `Vue` 一样能将科学与艺术完美的结合。

## 后续的工作

为了跟随 Vue 2.0, Muse 以 2.0 版本为基础，现在是 `alpha` 版，后续会不断完善。

* 修复现有的问题和合理化API

* 增加单元测试

* 增加更多快捷操作的api (简单的消息提示，alert, confirm 等等)

* 增加其它的功能性组件（Notification, Pagination 等等）

* 开发 weex 版的 muse



