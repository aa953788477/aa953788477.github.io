---
title: 关于 vue 弹窗组件的一些感想
date: 2016-09-08 08:34
tags: 
- vue
---


最近是用 vue 开发了一套组件库 [vue-carbon](https://myronliu347.github.io/vue-carbon/) , 在开发过程对对于组件化的开发有一些感想，于是开始记录下这些。

<!--more-->

弹窗组件一直是 web 开发中必备的，使用频率相当高，最常见的莫过于 alert，confirm，prompt .. 这些（曾经我们都会用alert来调试程序）, 不同的组件库对于弹窗的处理也是不一样的。在开发时需要考虑一下三点:

> 1. 进入和弹出的动画效果。
> 2. z-index 的控制
> 3. overlay 遮盖层

## 关于动画

vue 对于动画的处理相对简单，给组件加入css transition 动画即可

```html
<template>
<div class="modal" transition="modal-scale">
    <!--省略其它内容-->
</div>
</template>
<script>
// ...
</script>
<style>
.modal-scale-transition{
  transition: transform,opacity .3s ease;
}

.modal-scale-enter,
.modal-scale-leave {
    opacity: 0;
}

.modal-scale-enter {
  transform: scale(1.1);
}
.modal-scale-leave {
  transform: scale(0.8);
}
</style>
```

外部可以由使用者自行控制，使用 v-if 或是 v-show 控制显示

## z-index 的控制

关于z-index的控制，需要完成以下几点

> 1. 保证弹出框的 z-index 足够高能使 其再最外层
> 2. 后弹出的弹出框的 z-index 要比之前弹出的要高

要满足以上两点， 我们需要以下代码实现

```javascript
const zIndex = 20141223  // 先预设较高值

const getZIndex = function () {
    return zIndex++ // 每次获取之后 zindex 自动增加
}
```

然后绑定把 z-index 在组件上

```html
<template>
<div class="modal" :style="{'z-index': zIndex}" transition="modal-scale">
    <!--省略其它内容-->
</div>
</template>
<script>
export default {
    data () {
        return {
            zIndex: getZIndex()
        }
    }
}
</script>
```

## overlay 遮盖层的控制

遮盖层是弹窗组件中最难处理的部分, 一个完美的遮盖层的控制需要完成以下几点:

> 1. 遮盖层和弹出层之间的动画需要并行
> 2. 遮盖层的 z-index 要较小与弹出层
> 3. 遮盖层的弹出时需要组件页面滚动
> 4. 点击遮盖层需要给予弹出层反馈
> 5. 保证整个页面最多只能有一个遮盖层（多个叠在一起会使遮盖层颜色加深）


为了处理这些问题，也保证所有的弹出框组件不用每一个都解决，所以决定利用 `vue` 的 mixins 机制，将这些弹出层的公共逻辑封装层一个 `mixin` ，每个弹出框组件直接引用就好。

##  vue-popup-mixin

明确了上述所有的问题，开始开发 `mixin`, 首先需要一个 overlay (遮盖层组件) ;

```html
<template>
  <div class="overlay" @click="handlerClick" @touchmove="prevent" :style="style" transition="overlay-fade"></div>
</template>
<script>
export default {
  props: {
    onClick: {
      type: Function
    },
    opacity: {
      type: Number,
      default: 0.4
    },
    color: {
      type: String,
      default: '#000'
    }
  },
  computed: {
    style () {
      return {
        'opacity': this.opacity,
        'background-color': this.color
      }
    }
  },
  methods: {
    prevent (event) {
      event.preventDefault()
      event.stopPropagation()
    },
    handlerClick () {
      if (this.onClick) {
        this.onClick()
      }
    }
  }
}
</script>
<style lang="less">
.overlay {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  background-color: #000;
  opacity: .4;
  z-index: 1000;
}


.overlay-fade-transition {
  transition: all .3s linear;
  &.overlay-fade-enter,
  &.overlay-fade-leave {
    opacity: 0 !important;
  }
}
</style>
```

然后 需要一个 js 来管理 overlay 的显示和隐藏。

```javascript
import Vue from 'vue'
import overlayOpt from '../overlay'  // 引入 overlay 组件
const Overlay = Vue.extend(overlayOpt)

const getDOM = function (dom) {
  if (dom.nodeType === 3) {
    dom = dom.nextElementSibling || dom.nextSibling
    getDOM(dom)
  }
  return dom
}

// z-index 控制
const zIndex = 20141223  

const getZIndex = function () {
    return zIndex++ 
}
// 管理
const PopupManager = {
  instances: [],  // 用来储存所有的弹出层实例
  overlay: false,
  // 弹窗框打开时 调用此方法
  open (instance) {
    if (!instance || this.instances.indexOf(instance) !== -1) return
    
    // 当没有遮盖层时，显示遮盖层
    if (this.instances.length === 0) {
      this.showOverlay(instance.overlayColor, instance.overlayOpacity)
    }
    this.instances.push(instance) // 储存打开的弹出框组件
    this.changeOverlayStyle() // 控制不同弹出层 透明度和颜色
    
    // 给弹出层加上z-index
    const dom = getDOM(instance.$el)
    dom.style.zIndex = getZIndex()
  },
  // 弹出框关闭方法
  close (instance) {
    let index = this.instances.indexOf(instance)
    if (index === -1) return
    
    Vue.nextTick(() => {
      this.instances.splice(index, 1)
      
      // 当页面上没有弹出层了就关闭遮盖层
      if (this.instances.length === 0) {
        this.closeOverlay()
      }
      this.changeOverlayStyle()
    })
  },
  showOverlay (color, opacity) {
    let overlay = this.overlay = new Overlay({
      el: document.createElement('div')
    })
    const dom = getDOM(overlay.$el)
    dom.style.zIndex = getZIndex()
    overlay.color = color
    overlay.opacity = opacity
    overlay.onClick = this.handlerOverlayClick.bind(this)
    overlay.$appendTo(document.body)

    // 禁止页面滚动
    this.bodyOverflow = document.body.style.overflow
    document.body.style.overflow = 'hidden'
  },
  closeOverlay () {
    if (!this.overlay) return
    document.body.style.overflow = this.bodyOverflow
    let overlay = this.overlay
    this.overlay = null
    overlay.$remove(() => {
      overlay.$destroy()
    })
  },
  changeOverlayStyle () {
    if (!this.overlay || this.instances.length === 0) return
    const instance = this.instances[this.instances.length - 1]
    this.overlay.color = instance.overlayColor
    this.overlay.opacity = instance.overlayOpacity
  },
  // 遮盖层点击处理，会自动调用 弹出层的 overlayClick 方法
  handlerOverlayClick () {
    if (this.instances.length === 0) return
    const instance = this.instances[this.instances.length - 1]
    if (instance.overlayClick) {
      instance.overlayClick()
    }
  }
}

window.addEventListener('keydown', function (event) {
  if (event.keyCode === 27) { // ESC
    if (PopupManager.instances.length > 0) {
      const topInstance = PopupManager.instances[PopupManager.instances.length - 1]
      if (!topInstance) return
      if (topInstance.escPress) {
        topInstance.escPress()
      }
    }
  }
})

export default PopupManager
```

最后再封装成一个 mixin

```javascript
import PopupManager from './popup-manager'

export default {
  props: {
    show: {
      type: Boolean,
      default: false
    },
    // 是否显示遮盖层
    overlay: {
      type: Boolean,
      default: true
    },
    overlayOpacity: {
      type: Number,
      default: 0.4
    },
    overlayColor: {
      type: String,
      default: '#000'
    }
  },
  // 组件被挂载时会判断show的值开控制打开
  attached () {
    if (this.show && this.overlay) {
      PopupManager.open(this)
    }
  },
  // 组件被移除时关闭
  detached () {
    PopupManager.close(this)
  },
  watch: {
    show (val) {
      // 修改 show 值是调用对于的打开关闭方法
      if (val && this.overlay) {
        PopupManager.open(this)
      } else {
        PopupManager.close(this)
      }
    }
  },
  beforeDestroy () {
    PopupManager.close(this)
  }
}

```

## 使用

以上所有的代码就完成了所有弹出层的共有逻辑， 使用时只需要当做一个mixin来加载即可

```html
<template>
  <div class="dialog"
    v-show="show"
    transition="dialog-fade">
    <div class="dialog-content">
      <slot></slot>
    </div>
  </div>
</template>

<style>
  .dialog {
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    position: fixed;
    width: 90%;
  }

  .dialog-content {
    background: #fff;
    border-radius: 8px;
    padding: 20px;
    text-align: center;
  }

  .dialog-fade-transition {
    transition: opacity .3s linear;
  }

  .dialog-fade-enter,
  .dialog-fade-leave {
    opacity: 0;
  }
</style>

<script>
import Popup from '../src'

export default {
  mixins: [Popup],
  methods: {
    // 响应 overlay事件
    overlayClick () {
      this.show = false
    },
    // 响应 esc 按键事件
    escPress () {
      this.show = false
    }
  }
}
</script>
```

