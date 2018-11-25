---
title: 关于 Vue Mixin 和 Directive 的使用
date: 2017-01-31 09:35
tags: 
- vue
---

使用 vue 组件化开发已经很长时间，组件化的意义就是把在多个页面中共有的逻辑（显示/功能）封装，在多个页面或者多个项目中重复使用以提高开发效率。

但组件中共有的逻辑如何重用呢？例如： 我们再开发一个 `popover` 组件时都会有如下功能：

1. 点击外部时关闭弹出
2. 滚动条滚动或者窗口大小发生改变时重新定位

这里我们需要监听三个事件，一个是外部被点击事件，滚动条滚动，窗口大小发生改变，这些功能都很可能在别的组件中被使用，我们要将这些共有功能封装起来。

`Vue` 中提供了 **mixin**（混合） 和 **directive** 两种方式用于封装多个组件内的共有功能。

<!--more-->

## Mixin（混合）

> 混合是一种灵活的分布式复用 Vue 组件的方式。混合对象可以包含任意组件选项。以组件使用混合对象时，所有混合对象的选项将被混入该组件本身的选项。

> 当组件和混合对象含有同名选项时，这些选项将以恰当的方式混合。比如，同名钩子函数将混合为一个数组，因此都将被调用。另外，混合对象的 钩子将在组件自身钩子 之前 调用 ：

> 值为对象的选项，例如 methods, components 和 directives，将被混合为同一个对象。 两个对象键名冲突时，取组件对象的键值对。

`popover` 监听滚动和window大小变化功能，可以使用 `mixin` 完成

```javascript
// scroll.js
export default {
  props: {
    scroller: {
      type: [HTMLDocument, Element, Window],
      default () {
        return window
      }
    }
  },
  mounted () {
    this.$bindScroll()
  },
  methods: {
    $bindScroll () {
      if (!this.scroller) return
      this._handleScroll = (e) => {
        if (this.onScroll) this.onScroll()
      }
      this.scroller.addEventListener('scroll', this._handleScroll)
    },
    $unbindScroll (scroller) {
      scroller = scroller || this.scroller
      if (this._handleScroll) scroller.removeEventListener('scroll', this._handleScroll)
    }
  },
  beforeDestroy () {
    this.$unbindScroll()
  },
  watch: {
    scroller (scroller, oldScroller) {
      if (scroller === oldScroller) return
      this.$unbindScroll(oldScroller)
      this.$bindScroll(scroller)
    }
  }
}
```

```javascript
// resize.js
export default {
  mounted () {
    this.$bindResize()
  },
  methods: {
    $bindResize () {
      this._handleResize = (e) => {
        if (this.onResize) this.onResize()
      }
      window.addEventListener('resize', this._handleResize)
    },
    $unBindResize () {
      if (this._handleResize) window.removeEventListener('resize', this._handleResize)
    }
  },
  beforeDestroy () {
    this.$unBindResize()
  }
}
```

使用时，引入实现对应的方法即可

```javascript
// popover.js
import Scroll from 'scroll'
import Resize from 'resize'

export default {
    mixins: [Scroll, Resize],
    methods: {
        onScroll () {
            // 滚动时处理
        },
        onResize () {
            // 窗口大小变化时处理
        }
    }
}
```

同样 `scroll` 和 `resize` 也可以再其它组件中使用，复杂的项目中可以使用 mixin 将组件内共有的功能封装，但是要注意保证每个 mixin 处理的功能尽量单一，方便组件内部根据需要集成相应的功能。

## Directive（指令）

> 在 Vue2.0 里面，代码复用的主要形式和抽象是组件——然而，有的情况下,你仍然需要对纯 DOM 元素进行底层操作,这时候就会用到自定义指令。

例如: 需要让输入框根据参数自动 `focus` 和 `blur` 

```javascript
// focus.js
export default {
  bind (el, binding) {
    if (binding.value) el.focus()
  },
  update (el, binding) {
    if (!binding.oldValue && binding.value) el.focus()
  }
}
```

```html
<template>
    <input v-focus="isFocus"/>
</template>
<script>
import focus from 'focus'
export default {
  data () {
    return {
      isFocus: false
    }
  },
  directives: {
    focus
  }
}
</script>
```

指令中有很多钩子函数，可以根据实际情况实现

>* **bind**: 只调用一次，指令第一次绑定到元素时调用，用这个钩子函数可以定义一个在绑定时执行一次的初始化动作。
>* **inserted**: 被绑定元素插入父节点时调用（父节点存在即可调用，不必存在于 document 中）。
>* **update** : 被绑定元素所在的模板更新时调用，而不论绑定值是否变化。通过比较更新前后的绑定值，可以忽略不必要的模板更新（详细的钩子函数参数见下）。
>* **componentUpdated**: 被绑定元素所在模板完成一次更新周期时调用。
>* **unbind**: 只调用一次， 指令与元素解绑时调用。

每个钩子函数都会有如下参数：

>* **el**: 指令所绑定的元素，可以用来直接操作 DOM 。
>   * **binding**: 一个对象，包含以下属性：
>   * **name**: 指令名，不包括 v- 前缀。
>   * **value**: 指令的绑定值， 例如： v-my-directive="1 + 1", value 的值是 2。
>   * **oldValue**: 指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
>   * **expression**: 绑定值的字符串形式。 例如 v-my-directive="1 + 1" ， expression 的值是 "1 + 1"。
>   * **arg**: 传给指令的参数。例如 v-my-directive:foo， arg 的值是 "foo"。
>   * **modifiers**: 一个包含修饰符的对象。 例如： v-my-directive.foo.bar, 修饰符对象 modifiers 的值是 { foo: true, bar: true }。
>* **vnode**: Vue 编译生成的虚拟节点，查阅 VNode API 了解更多详情。
>* **oldVnode**: 上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。

一般在需要 DOM 操作时我们都需要使用自定义指令的方式去实现，当然一些特殊的事件监听也可以使用指令，例如上述中的监听外部点击事件：

```javascript
/**
 * element https://github.com/ElemeFE/element
 * clickoutside.js
 */
const clickoutsideContext = '@@clickoutsideContext'

export default {
  bind (el, binding, vnode) {
    const documentHandler = function (e) {
      if (!vnode.context || el.contains(e.target)) return
      if (binding.expression) {
        vnode.context[el[clickoutsideContext].methodName](e)
      } else {
        el[clickoutsideContext].bindingFn(e)
      }
    }
    el[clickoutsideContext] = {
      documentHandler,
      methodName: binding.expression,
      bindingFn: binding.value
    }
    setTimeout(() => {
      document.addEventListener('click', documentHandler)
    }, 0)
  },

  update (el, binding) {
    el[clickoutsideContext].methodName = binding.expression
    el[clickoutsideContext].bindingFn = binding.value
  },

  unbind (el) {
    document.removeEventListener('click', el[clickoutsideContext].documentHandler)
  }
}
```

```html
<template>
<div v-clickoutside="handleClickOutSide"></div>
</template>
<script>
import clickoutside from 'clickoutside'
export default {
  methods: {
    handleClickOutSide () {
        // 当外部被点击时调用
    }
  },
  directives: {
    clickoutside
  }
}
</script>
```

虽然 directive 和 mixin 都可用于封装组件中的公共功能，但是 directive 更趋向于有dom操作时使用，而mixin可以承担更复杂的功能封装。

以上代码，都在 [muse-ui](github.com/museui/muse-ui) 中实践

### 参考

[Vue 混合](https://cn.vuejs.org/v2/guide/mixins.html)
[Vue 自定义指令](https://cn.vuejs.org/v2/guide/custom-directive.html)



