---
title: 最简单的webapp开发
date: 2016-05-18 08:20:24
tags:
- webapp
- 移动端
- HTML5
---

之前我又想写过关于 [移动端页面开发](http://liuyy.coding.me/2016/05/14/webapp/webapp_mobile_page)，这篇则整理一下webapp开发需要的一些最简单的条件，以最简单的方式开发一个webapp；
<!--more-->

## 基础dom操作库

首先对于页面上的交互还是以dom操作为主，虽说现在javascript的dom api已经很好用了，但是对于用惯jquery的人来说还是很不方便，所以我选择 **zepto**，它的api和jquery完全一致，但是更为轻量。

## 路由处理

再手机端上页面切换会出现白屏，也无法做转场动画，为了坚决这些问题，我们需要开发一个单页面应用程序（ **SPA** ），通过解析url #后面的路径（#后面路径转换不会产生页面条件）加载不同的dom，这里也可以加上转场的动画，以获取和原生一样的用户体验。

[director.js](https://github.com/flatiron/director) 是最纯粹的路由注册/解析器，它在不刷新页面的情况下，利用“#”符号组织不同的URL路径，并根据不同的URL路径来匹配不同的回调方法。director.js不仅可以应用在客户端，在使用node.js的后台，它也能够实现前面说的后端路由功能。

## 缓存

移动端的网络需要考虑流量的消耗，不必要的重复请求是很粗糙的，所以很移动端需要大量使用缓存，来获取更好的用户体验。html5提供了**webstorage**机制储存数据，需要加一些简单的封装就可以使用了 [js缓存封装](http://liuyy.coding.me/2016/05/01/js%E7%BC%93%E5%AD%98%E5%B0%81%E8%A3%85/)

```javascript
localStorage.setItem(key, value);
localStorage.getItem(key);
localStorage.removeItem(key);
```

## 手势事件

移动端需要监测用户不同的手势来做相应的处理，例如：微信上列表上长按显示操作菜单，滑动返回。。。。，我们可以通过监测touch事件来判断用户的手势，touch事件有以下几种：

> touchstart     手指接触屏幕事件
> touchmove      手指再屏幕上移动事件
> touchend       手指离开屏幕事件
> touchcancel    非正常离开屏幕事件（例如在触摸时，别的应用打开，或者程序意外退出。。。）

简单的手势用以上几种事件配合就可以处理了，如果手势相对复杂，可以使用 [touch.js](http://touch.code.baidu.com/), 百度开发的手势事件库，api相当简单

```javascript

touch.on('.target', 'swipeleft swiperight', function(ev){
    console.log("you have done", ev.type);
});

```

## 滚动条

再android4.0以前手机上是不支持div滚动条的，很多的应用都引进了 [iscoll.js](http://cubiq.org/iscroll-5)，用js模拟滚动条，再4.0 以后有了div滚动条，可以使用原生滚动。但是在长列表渲染时，div滚动条会有闪白的情况，于是，大家就都使用，iscoll和div滚动切换的方式，再较低版本例如android 4.4之前用iscoll 4.4以上用原生滚动条。

## 下拉刷新和infinite-scroll

下拉刷新和无限滚动式手机端的必不可少的功能，js需要监听滚动和touch事件来实现这两个功能; [jQuery模拟原生态App上拉刷新下拉加载效果代码](http://justcoding.iteye.com/blog/2215557)


## 现有框架解决方案

### app.js

[App.js](http://www.peablog.com/project/appjs/) 的目标是为移动webapp提供一个健壮良好的开端，处理常见的功能并且兼容其他流行的JS库。提供良好的路由控制，有controller的mvc设计，也可以良好的兼容其它的框架，缺点是组件相对较少，但是对于简单的webapp还是没有问题的

### framework7

[Framework7](http://framework7.taobao.org/) 是一个开源免费的框架可以用来开发混合移动应用（原生和HTML混合）或者开发 iOS & Android 风格的WEB APP。也可以用来作为原型开发工具，可以迅速创建一个应用的原型。

framework7 组件丰富而且可以在android和ios两种风格建切换

## 打包

webapp需要调用原生方法，例如摄像头，通知等等，还需要将webapp打包成真正的apk或者app。这些需要通过一些工具完成;

### Phonegap

[phonegap](http://www.phonegap100.com/) 是一个跨平台的移动app开发框架，可以把html css js写的页面打包成跨平台的可以安装的移动app，并且可以调用原生的几乎所有的功能，比如摄像头，联系人，加速度等。

### HBuilder

[HBuilder](http://www.dcloud.io/) 是国产的混合是开发解决方案，包括有HBuilder编辑器 H5+规范调用原生api， mui框架，native.js与原生通信，它提供云打包以json格式配置app，再性能不错，经自己的比较再同样的程序的性能用Hbuilder更加。
