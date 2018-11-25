---
title: vue 开发波纹点击特效组件
date: 2016-09-18 16:34
tags: 
- vue
---

最近在使用 vue2 做一个新的 material ui 库，波纹点击效果在 material design 中被多次使用到，于是决定把它封装成一个公共的组件，使用时直接调用就好啦。

<!--more-->

## 开发之前的思考

常见的波纹点击效果的实现方式是监听元素的 mousedown 事件，在元素内部创建一个 **波纹元素** ，并调整元素的 `transform: scale(0);` 到 `transform: scale(1);`, 通过计算点击的位置来设置 **波纹元素** 的大小和位置，以达到波纹扩散的效果。

我将组件分为两个部分， `circleRipple.vue` 和 `TouchRipple.vue` 各自实现不同的功能

1. `circleRipple.vue` 波纹扩散组件，完成波纹扩散的效果
2. `TouchRipple.vue` 监听 `mouse` 和 `touch` 相关事件，控制  `circleRipple` 的显示，位置。

## circleRipple.vue

`circleRipple` 需要完成波纹扩展的效果，而且可以从外部控制它的大小和位置, 所以利用 `vue` 的 `transition` 动画完成效果, 提供 `mergeStyle` 、 `color` 、`opacity` 参数来从外部控制它的样式。实现代码如下。

```html
<template>
  <transition name="mu-ripple">
    <div class="mu-circle-ripple" :style="styles"></div>
  </transition>
</template>

<script>
import {merge} from '../utils'
export default {
  props: {
    mergeStyle: {
      type: Object,
      default () {
        return {}
      }
    },
    color: {
      type: String,
      default: ''
    },
    opacity: {
      type: Number
    }
  },
  computed: {
    styles () {
      return merge({}, {color: this.color, opacity: this.opacity}, this.mergeStyle)
    }
  }
}
</script>

<style lang="less">
@import "../styles/import.less";
.mu-circle-ripple{
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  pointer-events: none;
  user-select: none;
  border-radius: 50%;
  background-color: currentColor;
  background-clip: padding-box;
  opacity: 0.1;
}

.mu-ripple-enter-active, .mu-ripple-leave-active{
  transition: transform 1s @easeOutFunction, opacity 2s @easeOutFunction;
}

.mu-ripple-enter {
  transform: scale(0);
}

.mu-ripple-leave-active{
  opacity: 0 !important;
}
</style>
```

> vue2 对于动画方面做了比较大的修改，除了把指令换成组件外，它还可以完成更复杂的动画效果，具体可以看这里 [vue2 transition](http://rc.vuejs.org/guide/transitions.html)

## TouchRipple.vue

`TouchRipple` 需要控制 `circleRipple` 的显示。完成以下内容：

1. 监听 `mouse` 和 `touch` 相关事件， 控制 `circleRipple` 的显示。
2. 通过点击事件 event 对象， 计算出 `circleRipple` 的大小和位置
3. 如果频繁点击可能出现多个 `circleRipple`

### 首先，基本模板 + 数据模型

```html
<template>
  <!--最外层用div包裹-->
  <div @mousedown="handleMouseDown" @mouseup="end()" @mouseleave="end()" @touchstart="handleTouchStart"  @touchend="end()" @touchcancel="end()">
    <!--外层包裹防止波纹溢出-->
    <div :style="style" ref="holder">
      <!--多个波纹用 v-for 控制-->
      <circle-ripple :key="ripple.key" :color="ripple.color" :opacity="ripple.opacity" :merge-style="ripple.style" v-for="ripple in ripples"></circle-ripple>
    </div>
    <!--利用slot分发实际内容-->
    <slot></slot>
  </div>
</template>

<script>
import circleRipple from './circleRipple'
export default {
  props: {
    // 是否从中间扩散，设为false会从点击处扩散
    centerRipple: {
      type: Boolean,
      default: true
    },
    // 外层包裹的样式
    style: {
      type: Object,
      default () {
        return {
          height: '100%',
          width: '100%',
          position: 'absolute',
          top: '0',
          left: '0',
          overflow: 'hidden'
        }
      }
    },
    // 波纹颜色
    color: {
      type: String,
      default: ''
    },
    // 波纹透明度
    opacity: {
      type: Number
    }
  },
  data () {
    return {
      nextKey: 0, // 记录下一个波纹元素的key值， 相当于uuid，不设置的话会使动画失效
      ripples: [] // 波纹元素参数数组
    }
  },
  mounted () {
    this.ignoreNextMouseDown = false // 防止既有 touch 又有 mouse点击的情况
  },
  methods: {
    start (event, isRippleTouchGenerated) {
      // 开始波纹效果
    },
    end () {
      // 结束波纹效果
    },
    handleMouseDown (event) {
      // 监听 鼠标单击
    },
    handleTouchStart (event) {
      // 监听 touchstart 方法
    }
  },
  components: {
    'circle-ripple': circleRipple
  }
}
</script>

```

### 开始和结束波纹效果

增加一个波纹元素只需要在 **ripple** 增加一个 object 即可，不同的是当需要从点击处扩展时，需要计算一下波纹元素的大小和位置。

```javascript
{
  // isRippleTouchGenerated 是否是touch 事件开始的
  start (event, isRippleTouchGenerated) {
    // 过滤 touchstart 和 mousedown 同时存在的情况
    if (this.ignoreNextMouseDown && !isRippleTouchGenerated) {
      this.ignoreNextMouseDown = false
      return
    }
    
    // 添加一个 波纹元素组件
    this.ripples.push({
      key: this.nextKey++, 
      color: this.color,
      opacity: this.opacity,
      style: this.centerRipple ? {} : this.getRippleStyle(event) // 不是从中心扩展的需要计算波纹元素的位置
    })
    this.ignoreNextMouseDown = isRippleTouchGenerated
 },
 end () {
   if (this.ripples.length === 0) return
   this.ripples.splice(0, 1) // 删除一个波纹元素
   this.stopListeningForScrollAbort() // 结束 touch 滚动的处理
  }
}
```

因为 vue2 基于 Virtual DOM 的， 所以如果没有 `key` 在增加一个元素又同时删除一个元素的时候，dom tree并没有发生变化，是不会产生动画效果的。

### 监听 mousedown 和 touchstart

mousedown 和 touchstart 处理上会有所不同，但都是用来启动波纹效果的， touch涉及到多点点击的问题，我们一般取第一个即可。

```javascript
{
    handleMouseDown (event) {
      // 只监听鼠标左键的点击
      if (event.button === 0) {
        this.start(event, false)
      }
    },
    handleTouchStart (event) {
      event.stopPropagation() // 防止多个波纹点击组件嵌套
      if (event.touches) {
        this.startListeningForScrollAbort(event) // 启动 touchmove 触发滚动处理
        this.startTime = Date.now()
      }
      this.start(event.touches[0], true)
    }
}
```

### touchmove控制

当发生touchMove事件是需要判断是否，移动的距离和时间，然后结束小波纹点击小姑

```javascript
{
  // touchmove 结束波纹控制
  stopListeningForScrollAbort () {
    if (!this.handleMove) this.handleMove = this.handleTouchMove.bind(this)
    document.body.removeEventListener('touchmove', this.handleMove, false)
  },
  startListeningForScrollAbort (event) {
    this.firstTouchY = event.touches[0].clientY
    this.firstTouchX = event.touches[0].clientX
    document.body.addEventListener('touchmove', this.handleMove, false)
  },
  handleTouchMove (event) {
    const timeSinceStart = Math.abs(Date.now() - this.startTime)
    if (timeSinceStart > 300) {
      this.stopListeningForScrollAbort()
      return
    }
    const deltaY = Math.abs(event.touches[0].clientY - this.firstTouchY)
    const deltaX = Math.abs(event.touches[0].clientX - this.firstTouchX)
    // 滑动范围在 > 6px 结束波纹点击效果
    if (deltaY > 6 || deltaX > 6) this.end()
  }
}
```

### 计算波纹的位置和大小

需要从点击处扩散的波纹效果，需要计算波纹元素的大小和位置

```javascript
{
  getRippleStyle (event) {
    let holder = this.$refs.holder
    //  这个方法返回一个矩形对象，包含四个属性：left、top、right和bottom。分别表示元素各边与页面上边和左边的距离。
    let rect = holder.getBoundingClientRect() 
    // 获取点击点的位置
    let x = event.offsetX
    let y
    if (x !== undefined) {
      y = event.offsetY
    } else {
      x = event.clientX - rect.left
      y = event.clientY - rect.top
    }
    // 获取最大边长
    let max
    if (rect.width === rect.height) {
      max = rect.width * 1.412
    } else {
      max = Math.sqrt(
        (rect.width * rect.width) + (rect.height * rect.height)
      )
    }
    const dim = (max * 2) + 'px'
    return {
      width: dim,
      height: dim,
      // 通过margin控制波纹中心点和点击点一致
      'margin-left': -max + x + 'px',
      'margin-top': -max + y + 'px'
    }
  }
}
```

## 使用

由于 `touchRipple` 内部都是 **position:absolute** 布局，使用时，需要在外部加上 **position:relative** 

```html
// listItem.vue
<a :href="href" @mouseenter="hover = true" @mouseleave="hover = false" @touchend="hover = false"
    @touchcancel="hover = false" class="mu-item-wrapper" :class="{'hover': hover}">
    <touch-ripple class="mu-item" :class="{'mu-item-link': link}" :center-ripple="false">
      <div class="mu-item-media">
        <slot name="media"></slot>
      </div>
      <div class="mu-item-content">
        // ...
      </div>
    </touch-ripple>
</a>
<style>

.mu-item-wrapper {
    display: block;
    color: inherit;
    position: relative;
}
</style>
```

## 最后

到这点击波纹组件就开发完了， 这些代码借鉴了 [keen-ui](https://github.com/JosephusPaye/Keen-UI) 和 [material-ui](http://www.material-ui.com/) 的实现方式。

