---
title: 移动web开发准备工作
date: 2015-10-13 19:00
status: public
---

## meta设置
### html5移动端页面窗口自适应设备宽度，防止用户缩放页面
```html
<meta name="viewport" content="width=device-width,initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no,minimal-ui">
```
### 忽略将页面中的数字识别为电话号码
```html
<meta name="format-detection" content="telephone=no" />
```
### 忽略Android平台中对邮箱地址的识别
```html
<meta name="format-detection" content="email=no" />
```
### 当网站添加到主屏幕快速启动方式，可隐藏地址栏，仅针对ios的safari，将网站添加到主屏幕快速启动方式，
```html
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
```
### 完整的html模板
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" name="viewport">
<meta content="yes" name="apple-mobile-web-app-capable">
<meta content="black" name="apple-mobile-web-app-status-bar-style">
<meta content="telephone=no" name="format-detection">
<meta content="email=no" name="format-detection">
<title>标题</title>
<link rel="stylesheet" href="index.css">
</head>

<body>
这里开始内容
</body>

</html>
```
* * *
## css reset内容
在移动端中需要对浏览器做一些特殊的处理，以保证样式的一致性。
首先我们导入[Normalize.css](http://necolas.github.io/normalize.css/)的代码，让后再加上移动端的特殊处理.
```css
html{
    font-family:Helvetica;
    font-size:10px;/*设置10px为1em*/
    -webkit-tap-highlight-color: rgba(0,0,0,0);/*去掉触摸遮盖层*/
    -webkit-user-modify:read-write-plaintext-only;
    -webkit-user-select: none;/*禁止用户选择文字*/
	-webkit-overflow-scrolling: touch;/*滚动touch*/
}
/*设置所有的盒子元素*/
*{
    box-sizing: border-box;
}
```
## css技巧
### 消除闪屏
```css
/*设置内嵌的元素在 3D 空间如何呈现：保留 3D*/
-webkit-transform-style: preserve-3d;
/*（设置进行转换的元素的背面在面对用户时是否可见：隐藏）*/
-webkit-backface-visibility: hidden;
```
### 清除浮动
```css
.clearfix:before,
.clearfix:after {
    content: '\0020';
    display: block;
    overflow: hidden;
    visibility: hidden;
    width: 0;
    height: 0;
}
.clearfix:after { clear: both }
.clearfix { *zoom: 1 }
```
###部分android系统中元素被点击时产生的边框怎么去掉
```css
a,button,input,textarea{
-webkit-tap-highlight-color: rgba(0,0,0,0;)
-webkit-user-modify:read-write-plaintext-only;
}
```
### 开启硬件加速
```css
.css {
   -webkit-transform: translate3d(0, 0, 0);
   -moz-transform: translate3d(0, 0, 0);
   -ms-transform: translate3d(0, 0, 0);
   transform: translate3d(0, 0, 0);
}
```
### 如何阻止windows Phone的默认触摸事件
```css
html{-ms-touch-action: none;}/* 禁止winphone默认触摸事件 */
```
## js注意的地方
* 移动端上的点击事件会延迟300ms，可以使用touchstart代替（一般使用fastclick库）
