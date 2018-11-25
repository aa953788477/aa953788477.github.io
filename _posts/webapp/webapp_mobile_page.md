---
title: 关于webapp开发——移动端页面
date: 2016-05-14 20:24:39
tags:
- webapp
- 移动端
- HTML5
---

去年的时候有写过一篇关于移动web开发的文章，经过一年的webapp开发，对于移动端页面的开发又有了一个新的看法。总结一下怎么做好一个webapp，首先我们从移动端页面开始吧。

## viewport

简单来说viewport就是浏览器用来显示网页的部分(也可能是一个app中的webview)，但是viewport的大小并不局限于浏览器可视区域，一般来说viewport的大小要比浏览器可视区域要大，这是因为考虑到移动设备的分辨率相对于桌面电脑来说都比较小，所以为了能在移动设备上正常显示那些传统的为桌面浏览器设计的网站，移动设备上的浏览器都会把自己默认的viewport设为980px或1024px（也可能是其它值，这个是由设备自己决定的），但带来的后果就是浏览器会出现横向滚动条，因为浏览器可视区域的宽度是比这个默认的viewport的宽度要小的。下图列出了一些设备上浏览器的默认viewport的宽度。

![](http://images.cnitblog.com/blog/130623/201407/300958470402077.png)

<!--more-->

## meta设置

既然浏览器是用过 **viewport** 展示网页，所以我们要让手机端的网页不用通过缩放的方式来浏览，我们把viewport的大小设置同浏览器可是区域一样，并且禁止缩放网页。这些再 **head** 中加入 **meta** 设置完成。

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no,minimal-ui">
```

* width viewport的宽度
* initial-scale 初始化缩放比例
* minimum-scale 最小缩放比例
* maxinum-scale 最大缩放比例
* user-scalable  用户是否可以缩放
* minimal-ui ios7以上隐藏浏览器导航栏

## css reset

犹豫浏览器对内置标签的css默认属性不同，css reset就是为了统一不同浏览器的标签默认样式，这里我们就重复造轮子了，使用[normalize.css](http://necolas.github.io/normalize.css/), 另外不同元素的宽度计算不一样，需要加上这段代码

```css
*,
*:before,
*:after{
	box-sizing: border-box;
}
```

## 关于单位 px em 和 rem

我们的页面会在不同尺寸的设备上运行，当我们需要把整体字体大小加大时，一个一个修改的方法实在太笨了。所以在响应式页面中需要使用em或是rem作为单位,下面简单解释下em，和rem。

em和rem都是相对单位，像是这样

```css
html{
	font-size: 10px;
}
body{
	font-size: 14px;
	font-size: 1.4em;
	font-size: 1.4rem;
}
```

唯一不同时em是相对于父元素的大小，rem是相对于根元素(html元素)

## 关闭点击高亮

浏览器中默认可以的点击元素在点击之后都未有高亮显示，但是由于手机端会有300ms的点击延迟，所以最好关闭这种高亮，然后自己添加样式控制

```css
{
-webkit-tap-highlight-color:rgba(0, 0, 0, 0);
}
```

## 禁止选取文本和文本溢出处理

```css
.no-select{
	user-select: none;
	-webkit-user-select: none;
}
.ellipsis{
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
```

## ios回弹滚动和去除输入框阴影

```css
.scroll{
	overflow: auto;
	-webkit-overflow-scrolling: touch;
}
input,
textarea {
    border: 0; /* 方法1 */
    -webkit-appearance: none; /* 方法2 */
}
```
## 监测键盘弹出事件

```javascript
 var timer, windowInnerHeight;
 function eventCheck(e) {
       if (e) { //blur,focus事件触发的
            $('#dv').html('android键盘' + (e.type == 'focus' ? '弹出' : '隐藏') + '--通过' + e.type + '事件');
            if (e.type == 'click') {//如果是点击事件启动计时器监控是否点击了键盘上的隐藏键盘按钮，没有点击这个按钮的事件可用，keydown中也获取不到keyCode值
                setTimeout(function () {//由于键盘弹出是有动画效果的，要获取完全弹出的窗口高度，使用了计时器
                    windowInnerHeight = window.innerHeight;//获取弹出android软键盘后的窗口高度
                    timer = setInterval(function () { eventCheck() }, 100);
                }, 500);
            }
            else clearInterval(timer);
        }
        else { //计时器执行的，需要判断窗口可视高度，如果改变说明android键盘隐藏了
            if (window.innerHeight > windowInnerHeight) {
                clearInterval(timer);
                $('#dv').html('android键盘隐藏--通过点击键盘隐藏按钮');
            }
        }
    }
    $('#txt').click(eventCheck).blur(eventCheck);

```
## click 300ms延迟

移动设备为了监听双击缩放viewport事件，再touchend之后要等待300ms来判断是否双击在处罚事件。[fastclick]() 就是为了解决这个问题而生的，它会再touchend之后手动触发click事件，然后阻止原来的click事件。

```javascript
FastClick.attach( document.body );
```

## css动画 GPU加速

设置3d相关属性开区GPU加速

```css
.animate{
-webkit-transform: translate3d(0, 0, 0);
-moz-transform: translate3d(0, 0, 0);
-ms-transform: translate3d(0, 0, 0);
transform: translate3d(0, 0, 0);
}
```

## 消除动画闪屏

```css
.animate{
-webkit-backface-visibility: hidden;
-moz-backface-visibility: hidden;
-ms-backface-visibility: hidden;
backface-visibility: hidden;

-webkit-perspective: 1000;
-moz-perspective: 1000;
-ms-perspective: 1000;
perspective: 1000;
}
```

## 设备检测

最后，在某些特殊的情况，我们需要针对不同浏览器做特殊处理。

```javascript
// 这段代码引用自：https://github.com/binnng/device.js

var WIN = window;
var LOC = WIN["location"];
var NA = WIN.navigator;
var UA = NA.userAgent.toLowerCase();

function test(needle) {
  return needle.test(UA);
}

var IsTouch = "ontouchend" in WIN;
var IsAndroid = test(/android|htc/) || /linux/i.test(NA.platform + "");
var IsIPad = !IsAndroid && test(/ipad/);
var IsIPhone = !IsAndroid && test(/ipod|iphone/);
var IsIOS = IsIPad || IsIPhone;
var IsWinPhone = test(/windows phone/);
var IsWebapp = !!NA["standalone"];
var IsXiaoMi = IsAndroid && test(/mi\s+/);
var IsUC = test(/ucbrowser/);
var IsWeixin = test(/micromessenger/);
var IsBaiduBrowser = test(/baidubrowser/);
var IsChrome = !!WIN["chrome"];
var IsBaiduBox = test(/baiduboxapp/);
var IsPC = !IsAndroid && !IsIOS && !IsWinPhone;
var IsHTC = IsAndroid && test(/htc\s+/);
var IsBaiduWallet = test(/baiduwallet/);
```
