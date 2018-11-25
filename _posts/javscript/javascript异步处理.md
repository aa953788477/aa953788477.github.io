---
date: 2015-11-15 00:26
title: javascript异步处理
tags: javascript
---

javascript是单线程语言，所谓"单线程"，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。
这种方式比较好控制，不会存在像java那个多线程死锁等等复杂的问题，缺点也很明显，当一个任务执行较长时间时，后续的任务需要等待，会拖延整个程序的执行，使浏览器卡死。
为了解决这个问题，javascript将任务分为两种：同步（Synchronous）和异步（Asynchronous）;
<!--more-->
在浏览器端，如果需要执行比较耗时的任务，为了避免浏览器卡死，就需要异步执行，不影响其他任务，最常见的例子是ajax操作。
但是一旦使用异步，还要保证有些任务再异步任务之后执行，这个时候就需要特殊的处理了，以下介绍四中常见的处理异步的方法
## 一、回调函数
这个是最常见的处理方式，假设一个ajax请求，设定一个success方法再请求成功时回调
```javascript
$.ajax(
    url :'test',
    success : function(data){
    }
);
```
这样ajax请求不会阻塞其他程勋的正常运行
回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数。
## 二、事件监听
事件驱动模式。success任务在请求成功事件发生时执行。还是以ajax为例：
```javascript
function success(){
    console.log('the ajax successed');
}
$.on('success', success);//监听事件 如果发生执行success方法
$.ajax(
    url : 'test',
    success : function(){
        $.trigger('success');//出发success 事件
    }
);
```
这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以去耦合（Decoupling），有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。
## 三、发布/订阅
基于观察者模式，可以用新闻的方式来理解，例：
```javascript
$.subscribe('news', function(){
    console.log("我来看新闻");
}); //订阅新闻
$.ajax(
    url : 'getNews',
    success : function(){
        $.publish("news");//发布新闻
    }
);
```
当ajax请求完成之后会发布信号告诉所有的订阅者“我已经执行完了”，订阅者接收到后，会执行相应的操作
这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。
## 四、Promise对象
Promises对象是CommonJS工作组提出的一种规范，目的是为异步编程提供统一接口。
简单说，每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。就像下面这样:
```javascript
    start().then(step1).then(step2)
```
这样写的优点在于，回调函数变成了链式写法，程序的流程可以看得很清楚，而且有一整套的配套方法，可以实现许多强大的功能。
实现代码如下:
```javascript
/**
 * Promise.js
 * 异步处理的Promise对象
 */
;(function(Global, factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        module.export = factory();
    } else if (define && typeof define === 'function') {
        define(function() {
            return factory();
        });
    } else {
        if (!window.Promise) window.Promise = factory();
    }
})(window, function() {
    var Promise = function(startFunc) {
        this.state = 'pending';
        this.thenables = []; //存储接下来的步骤
        if(startFunc && typeof startFunc === 'function'){
            startFunc(this.resolve, this.reject);
        }
    };
    /**
     * 改变状态为已完成
     * @param  {[type]} value 传递的值
     * @return {[type]}       [description]
     */
    Promise.prototype.resolve = function(value) {
        if (this.state != 'pending') return;
        this.state = 'fulfilled';
        this.value = value;
        this._handleThen();
        return this;
    };
    /**
     * 改变状态为有异常
     * @param  {[type]} reason [description]
     * @return {[type]}        [description]
     */
    Promise.prototype.reject = function(reason) {
        if (this.state != 'pending') return;
        this.state = 'rejected';
        this.reason = reason;
        this._handleThen();
        return this;
    };
    Promise.prototype.then = function(onFulfilled, onRejected) {
        var thenable = {};
        if (typeof onFulfilled == 'function') {
            thenable.fulfill = onFulfilled;
        }
        if (typeof onRejected == 'function') {
            thenable.reject = onRejected;
        }
        /**
         * 如果状态不是pending就立即执行
         */
        if (this.state != 'pending') {
            if (setImmediate) {
                setImmediate(function() {
                    this._handleThen();
                }.bind(this));
            } else {
                setTimeout(function() {
                    this._handleThen();
                }.bind(this), 0);
            }
        }
        thenable.promise = new Promise();
        this.thenables.push(thenable);
        return thenable.promise;
    };
    Promise.prototype._handleThen = function() {
        if (this.state === 'pending') return;
        if (this.thenables.length) {
            for (var i = 0; i < this.thenables.length; i++) {
                var thenPromise = this.thenables[i].promise;
                var returnedVal;
                try {
                    switch (this.state) {
                        case 'fulfilled':
                            if (this.thenables[i].fulfill) {
                                returnedVal = this.thenables[i].fulfill(this.value);
                            } else {
                                thenPromise.resolve(this.value);
                            }
                            break;
                        case 'rejected':
                            if (this.thenables[i].reject) {
                                returnedVal = this.thenables[i].reject(this.reason);
                            } else {
                                thenPromise.reject(this.reason);
                            }
                            break;
                    }
                    if (returnedVal === null) {
                        this.thenables[i].promise.resolve(returnedVal);
                    /*判断返回值是否是Promise对象，如果值那么下个promise执行要绑定在返回值的then方法上*/
                    } else if (returnedVal instanceof Promise || typeof returnedVal.then === 'function') {
                        returnedVal.then(thenPromise.resolve.bind(thenPromise), thenPromise.reject.bind(thenPromise));
                    } else {
                        this.thenables[i].promise.resolve(returnedVal);
                    }
                } catch (e) {
                    thenPromise.reject(e);
                }
            }
            this.thenables = [];
        }
    };
    return Promise;
});
```
## 需要注意的地方
1. 异步回调函数都是在主任务执行完毕后执行,如果主任务卡死，那么回调函数将不会执行;
2. ajax的同步方法最好不要轻易使用，程序需要等待请求成功才会继续向下执行，如果请求时间过长浏览器会卡死；
3. 如果你想保证你的某些函数再主函数执行完后再执行可以使用setTimout(function(){}，0);