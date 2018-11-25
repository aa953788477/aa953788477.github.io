---
title: html5设备api
date: 2016-05-31 09:00:02
tags: HTML5
---

为了更好地为移动设备服务，HTML 5推出了一系列针对移动设备的API。

## Viewport

Viewport指的是网页的显示区域，也就是不借助滚动条的情况下，用户可以看到的部分网页大小，中文译为“视口”。正常情况下，viewport和浏览器的显示窗口是一样大小的。但是，在移动设备上，两者可能不是一样大小。

<!--more-->

一般来说viewport的大小要比浏览器可视区域要大，这是因为考虑到移动设备的分辨率相对于桌面电脑来说都比较小，所以为了能在移动设备上正常显示那些传统的为桌面浏览器设计的网站，移动设备上的浏览器都会把自己默认的viewport设为980px或1024px（也可能是其它值，这个是由设备自己决定的），但带来的后果就是浏览器会出现横向滚动条，因为浏览器可视区域的宽度是比这个默认的viewport的宽度要小的。下图列出了一些设备上浏览器的默认viewport的宽度。

![](http://images.cnitblog.com/blog/130623/201407/300958470402077.png)

我们在开发移动端页面的时候都会去设置viewport宽度

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no,minimal-ui">
```

上面代码指定，viewport的缩放规则是，缩放到当前设备的屏幕宽度（device-width），初始缩放比例（initial-scale）为1倍，禁止用户缩放（user-scalable）。

> * width viewport的宽度
> * initial-scale 初始化缩放比例
> * minimum-scale 最小缩放比例
> * maxinum-scale 最大缩放比例
> * user-scalable  用户是否可以缩放
> * minimal-ui ios7以上隐藏浏览器导航栏

## Geolocation API

Geolocation接口用于获取用户的地理位置。它使用的方法基于GPS或者其他机制（比如IP地址、Wifi热点、手机基站等）。

getCurrentPosition方法，用来获取用户的地理位置。使用它需要得到用户的授权，浏览器会跳出一个对话框，询问用户是否许可当前页面获取他的地理位置。必须考虑两种情况的回调函数：一种是同意授权，另一种是拒绝授权。如果用户拒绝授权，会抛出一个错误。

```javascript
navigator.geolocation.getCurrentPosition(function(event){
    console.log(event.coords.latitude + ', ' + event.coords.longitude);
}, function(event){
    console.log("Error code " + event.code + ". " + event.message);
});
```

成功后的函数的参数是一个event对象。event有两个属性：timestamp和coords。timestamp属性是一个时间戳，返回获得位置信息的具体时间。coords属性指向一个对象，包含了用户的位置信息，主要是以下几个值：

> coords.latitude：纬度
> coords.longitude：经度
> coords.accuracy：精度
> coords.altitude：海拔
> coords.altitudeAccuracy：海拔精度（单位：米）
> coords.heading：以360度表示的方向
> coords.speed：每秒的速度（单位：米）

## Vibration API

Vibration接口用于在浏览器中发出命令，使得设备振动。显然，这个API主要针对手机，适用场合是向用户发出提示或警告，游戏中尤其会大量使用。由于振动操作很耗电，在低电量时最好取消该操作。

```javascript
navigator.vibrate(1000); //震动 1s

navigator.vibrate([500, 300, 100]); //震动 500ms 等待 300ms 再震动 100ms
```

## Orientation

Orientation API用于检测手机的摆放方向（竖放或横放）。

一旦设备的方向发生变化，会触发deviceorientation事件，可以对该事件指定回调函数。

```javascript
window.addEventListener("deviceorientation", function(event){
    console.log(event.alpha);
	console.log(event.beta);
	console.log(event.gamma);
});
```

上面代码中，event事件对象有alpha、beta和gamma三个属性，它们分别对应手机摆放的三维倾角变化。要理解它们，就要理解手机的方向模型。当手机水平摆放时，使用三个轴标示它的空间位置：x轴代表横轴、y轴代表竖轴、z轴代表垂直轴。event对象的三个属性就对应这三根轴的旋转角度。

> alpha：表示围绕z轴的旋转，从0到360度。当设备水平摆放时，顶部指向地球的北极，alpha此时为0。
> beta：表示围绕x轴的旋转，从-180度到180度。当设备水平摆放时，beta此时为0。
> gramma：表示围绕y轴的选择，从-90到90度。当设备水平摆放时，gramma此时为0。

## DeviceMotionEvent

DeviceMotionEvent 用于检测设备加速度，例如， 摇一摇，重力感应等等, 下面这段代码用于监控手机晃动

```javascript
var SHAKE_THRESHOLD = 3000;
var last_update = 0;
var x = y = z = last_x = last_y = last_z = 0;
function init() {
    if (window.DeviceMotionEvent) {
        window.addEventListener('devicemotion', deviceMotionHandler, false);
    } else {
        alert('not support mobile event');
    }
}
function deviceMotionHandler(eventData) {
    var acceleration = eventData.accelerationIncludingGravity;
    var curTime = new Date().getTime();
    if ((curTime - last_update) > 100) {
        var diffTime = curTime - last_update;
        last_update = curTime;
        x = acceleration.x;
        y = acceleration.y;
        z = acceleration.z;
        var speed = Math.abs(x + y + z - last_x - last_y - last_z) / diffTime * 10000;
        if (speed > SHAKE_THRESHOLD) {
            alert("摇动了");
        }
        last_x = x;
        last_y = y;
        last_z = z;
    }
}
```

上面代码通过判断两次手机移动的距离和时间，来判断手机手否晃动。
