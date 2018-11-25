
---
date: 2016-09-01 15:47
status: public
title: 移动web开发中有用的css片段
tags: 
- css
- 移动web
---

在做移动 web 开发中有很多地方跟 PC 段是不一样的，需要不一样的 css reset， 或者使用flexbox布局，等等，这里就记录下移动 web 开发中有用的 css 片段。
<!--more-->
## css reset
移动web中reset 除了常规的 [Normalize.css](http://necolas.github.io/normalize.css/) 之外，还需要加上下面这些代码

```css
html{
    -webkit-tap-highlight-color: rgba(0,0,0,0);/*去掉触摸遮盖层*/
    -webkit-user-modify: read-write-plaintext-only;
    -webkit-user-select: none;/*禁止用户选择文字*/
}

/*设置所有盒子大小计算边框内*/
*
*:before,
*:after {
    box-sizing: border-box;
}

/*消除输入框的阴影和边框*/
input,textarea, select{
    -webkit-appearance: none;
    appearance: none;
    outline: none;
    border: none;
}
```

## 消除 transition 动画闪屏

移动 web 中常会使用到 *transition* 动画，在某些设备上会出现闪屏的现象，需要在样式加上下面这些

```css
.animate {
    /* .... */
    /*设置内嵌的元素在 3D 空间如何呈现：保留 3D*/
    -webkit-transform-style: preserve-3d;
    /*（设置进行转换的元素的背面在面对用户时是否可见：隐藏）*/
    -webkit-backface-visibility: hidden;
}
```

## 弹性滚动

ios 设备上设置弹性滚动属性，可以使滚动更加流畅

```css
.scollable{
    overflow: auto;
    -webkit-overflow-scrolling: touch;/*滚动touch*/
}
```

## 点击状态处理

reset 关闭了点击高亮（因为会延迟），需要使用伪元素实现点击高亮效果

```css
.active:before{
    content: '';
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    background-color: @color;
    background-repeat: no-repeat;
    background-position: center;
    background-size: 100% 100%;
    opacity: 0;
    pointer-events: none;
    -webkit-transition: opacity 600ms;
    transition: opacity 600ms;   /*加上动画显得更自然*/
}

.active:active:before{
    opacity: 1;
    -webkit-transition: opacity 150ms;
    transition: opacity 150ms;
}
```


## 1px边框处理

移动端会有很多的高清屏，导致 `border-width: 1px` 的边框显得特别粗，所以需要使用伪元素模拟边框，检测不同的屏幕然后缩小高度，使其像头发丝一样细.

```css
/*border 在底部*/
.has-border:after{
    content: '';
    position: absolute;
    right: 0;
    top: 0;
    left: auto;
    bottom: auto;
    width: 1px;
    height: 100%;
    background-color: rgba(0, 0, 0, .12);
    display: block;
    z-index: 15;
}
html.pixel-ratio-2 & {
    -webkit-transform: scaleX(0.5);
    transform: scaleX(0.5);
}
html.pixel-ratio-3 & {
    -webkit-transform: scaleX(0.33);
    transform: scaleX(0.33);
}
```

通过 js 检测 *devicePixelRatio* 然后 html 加上相应的样式

```javascript
// 处理retina屏幕显示效果
var classNames = []
var pixelRatio = window.devicePixelRatio || 1
classNames.push('pixel-ratio-' + Math.floor(pixelRatio))
if (pixelRatio >= 2) {
  classNames.push('retina')
}

var html = document.getElementsByTagName('html')[0]

classNames.forEach((className) => html.classList.add(className))
```


## 使用 fastclick 之后，label点击的问题

为了解决 *点击事件300ms延迟* 的问题, 都会引入 `fastclick` 库, 但是会发现当 label 内部有其它元素时不能给表单元素聚焦了。

```html
<label>
    <input type="text"/>
    <!--这个时候点击 span标签中的内容 input无法获取焦点-->
    <span>text</span>
</label>
```

需要使用 `pointer-event` 让 label中元素的点击事件都冒泡到 label 上再执行

```css
/*这里是示例代码， 实际使用最好制定 className*/
label *{
    pointer-events: none;
}
```


## 关于布局

移动端中 对于 flexbox  的支持已经很好，flexbox 是布局神器，像页面 上中下这种结构的布局很容易就实现了。

```html
<div class="page">
    <div class="header"></div>
    <div class="content"></div>
    <div class="footer"></div>
</div>
```

```css
.page{
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: flex-start;
}
.header,
.footer{
    width: 100%;
    height: 56px
}
.content{
    flex: 1;
    width: 100%;
}
```

另外阮一峰老师也写过 [flexbox 入门教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

