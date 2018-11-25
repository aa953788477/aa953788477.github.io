---
title: webapp 性能优化
date: 2016-05-29 20:18:24
tags:
- webapp
- 移动端
- HTML5
---

webapp 不像传统页面，它生命周期更长，在手机端上，硬件环境并没有pc上那么好。所以性能的优化尤为重要。 webapp的性能优化主要分为两个方面 **网络请求优化** 和 **页面渲染优化** ， 我们对于性能优化主要通过这连个方面来处理。

<!--more-->

## 压缩资源文件

我们在使用某些框架的时候（例如：JQuery）, 会发现有两个文件 **jquery.js** 和 **jquery.min.js**， **jquery.min.js** 的体积会小很多，这样在请求的时候会比较快。

所以在项目中使用到的资源文件（js、css、image）在发布之前需要进行压缩处理, 这里有些在线的压缩的工具 [在线压缩](http://tool.oschina.net/jscompress)， 如果使用写项目管理工具， 可以考虑用 gulp 请求。

```javascript
var gulp = require('gulp'),
    uglify = require('gulp-uglify'),
    rename = require('gulp-rename'),
    sourcemaps = require('gulp-sourcemaps'),
    minifycss = require('gulp-minify-css');

gulp.task('js', function(){
	return gulp.src('./test.js')
			.pipe(sourcemaps.init())
			.pipe(rename({suffix: '.min'}))
  			.pipe(uglify())
  			.pipe(sourcemaps.write('./'))
    		.pipe(gulp.dest('./'));
});

gulp.task('css', function(){
    return gulp.src('./test.css')
    .pipe(rename({suffix: '.min'}))
    .pipe(minifycss())
    .pipe(gulp.dest('/'));;
});

```

## 合并请求

开发过程中，维护一个很长的js或是css文件是一个很困难的事情，常常会吧它分为很多的小得js和css文件，但是如果按照传统的方式写script/link 标签加载，会多很多请求，有http 1请求的原因，每个请求都会有一个rtt（请求回路）时间，一旦请求数量增多，那么请求的时间也会变长。 在发布之前我们可以使用一些工具将这些资源文件合并为一个, 我们常使用gulp

```javascript
var gulp = require('gulp'),
    uglify = require('gulp-uglify'),
    rename = require('gulp-rename'),
    concat = require('gulp-concat'),
    sourcemaps = require('gulp-sourcemaps'),
    minifycss = require('gulp-minify-css');

gulp.task('js', function(){
	return gulp.src('./app/**/**.js') //app 下所有的js 文件
			.pipe(sourcemaps.init())
            .pipe(concat('./app.js')) //合并所有文件
            .pipe(gulp.dest('./'))
			.pipe(rename({suffix: '.min'}))
  			.pipe(uglify())
  			.pipe(sourcemaps.write('./'))
    		.pipe(gulp.dest('./'));
});

gulp.task('css', function(){
    return gulp.src('./app/**/*.css')
    .pipe(concat('./app.css'))          //app 下所有的 css文件
    .pipe(gule.dest('./'))         
    .pipe(rename({suffix: '.min'}))
    .pipe(minifycss())
    .pipe(gulp.dest('/'));;
});

```

## cdn 和 gzip

**CDN** 的全称是 **Content Delivery Network** ，即内容分发网络。其基本思路是尽可能避开互联网上有可能影响数据传输速度和稳定性的瓶颈和环节，使内容传输的更快、更稳定。通过在网络各处放置节点服务器所构成的在现有的互联网基础之上的一层智能虚拟网络，CDN系统能够实时地根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。其目的是使用户可就近取得所需内容，解决 Internet网络拥挤的状况，提高用户访问网站的响应速度。

**GZIP** 最早由 Jean-loup Gailly 和 Mark Adler 创建，用于UNⅨ系统的文件压缩。我们在Linux中经常会用到后缀为.gz的文件，它们就是GZIP格式的。现今已经成为Internet 上使用非常普遍的一种数据压缩格式，或者说一种文件格式。
HTTP协议上的GZIP编码是一种用来改进WEB应用程序性能的技术。大流量的WEB站点常常使用GZIP压缩技术来让用户感受更快的速度。这一般是指WWW服务器中安装的一个功能，当有人来访问这个服务器中的网站时，服务器中的这个功能就将网页内容压缩后传输到来访的电脑浏览器中显示出来.一般对纯文本内容可压缩到原大小的40%.这样传输就快了，效果就是你点击网址后会很快的显示出来.当然这也会增加服务器的负载. 一般服务器中都安装有这个功能模块的。

我们可以将静态的资源发布到cdn上，并开启 **gzip** 会大大的提高页面加载数据

## lazyLoad

webapp 首屏加载速度是一个重要的指标，那么怎么让我们首页最快速度加载出来呢？首页中我们能直接看到的只有一屏，那么剩下的我们可以做 lazyload， 先加加载能直接看到，让后异步延迟加载剩下的内容。

## 数据缓存

webapp 中有很多和后台的交互，首页加载的时候可以先使用上次的缓存数据，等请求成功后再刷新这部分的显示， 在一些短暂时间的重复请求，可以使用 **sessionStorage** 缓存这些数据， 使用 [一些手段](http://liuyy.coding.me/2016/05/01/js%E7%BC%93%E5%AD%98%E5%B0%81%E8%A3%85/) 设置缓存时间，短暂时间的内的重复请求就可以直接从缓存中获取数据了。

```javascript
//封装请求方法
function getData(url, params, onloadSuccess){
    var key = url; //使用 url + params 做key
    for(var str in params){
        key += str += params[str];
    }
    if(sessionStorage.getItem(key)){
        //如果有缓存直接从缓存中取
        onloadSuccess(JSON.stringify(sessionStorage.getItem(key)));
        reutrn ;
    }
    $.ajax({
        url: url,
        data: data,
        dataType: 'json',
        type: 'post',
        success: function(data){
            sessionStorage.setItem(key, data); //缓存数据
            onloadSuccess(data);
        }
    });
}
```

## css动画GPU加速消除闪屏

```css

.animate{
    -webkit-transform: translate3d(0, 0, 0);
    -moz-transform: translate3d(0, 0, 0);
    -ms-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
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

## js动画

有些动画需要一些js控制无法使用css动画的情况，我们可以考虑使用js动画来做。可以使用window.requestAnimationFrame.

**window.requestAnimationFrame()** 这个方法是用来在页面重绘之前，通知浏览器调用一个指定的函数，以满足开发者操作动画的需求。这个方法接收一个函数作为参数，改函数在重绘前调用。

它稳定性比 **setTimeout** 要好，但是并不是所有的浏览器都支持这个api， 我们需要一个兼容性代码处理。

```javascript
window.requestAnimationFrame = (function() {
    return window.requestAnimationFrame ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame ||
          window.oRequestAnimationFrame ||
          window.msRequestAnimationFrame ||
          function(callback) {
              return window.setTimeout(callback, 1000 / 60);
          };
})();
```

## 布局优化

PC 上有很多用float来做的布局，float的布局在需要更大的计算量，在页面渲染的时候严重影响到了加载速度，尽量考虑用 **inline** 和 **inline-block** 替代, **flex box** 是专为响应式设计布局方式，在移动端兼容性良好，可以考虑大范围使用，这里推荐下阮一峰大大的教程 [Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

## 优化页面上的dom数量

在mobile端，如果页面中的dom数量过多，在页面滚动的时候会出现卡顿，闪烁等等不良的情况，所以我们要删除一些页面上不必要的dom元素或者将一些dom元素作为延迟加载。

## 优化dom查询性能

先看一下这个部分的代码

```javascript
for(var i = 0; i< 100; i++){
    var a = $('.test');
}
```

上面的代码每次循环的时候都会去进行一次dom查询，显然是不必要的消耗，我们需要在dom查询之前缓存，避免不必要的dom操作。

```javascript
var a = $('.test');
for(var i = 0; i< 100; i++){
    // do somethings
}
```
