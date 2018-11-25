---
date: 2016-03-21 23:30
title: javascript模块化编程(一)
tags: javascript
---

随着web开发逐渐演变成web application之后，尤其是在web2.0依赖页面上的javascript脚本逐渐变多，嵌入网页的JavaiScript代码越来越庞大，越来越复杂。网页越来越像桌面程序，需要一个团队分工协作、进度管理、单元测试等等……开发者需要分模块，来管理前端代码开发。
<!--more-->
## 原始写法
模块就是实现特定功能的一组方法。只要把不同的函数（以及记录状态的变量）简单地放在一起，就算是一个模块。
```javascript
    var state = '';
    function method1(){
        //do somethings
    }
    function mehtod2(){
        //do somthings
    }
```
上面吧state和方法放在一起组成一个模块，要使用的是否，直接调用方法即可。但是这种做法缺点也很明显，吧，变量和方法都放在了全局变量里，造成了全局污染，一但模块数量变多，后面相同名称方法或是变量就会覆盖之前的，产生一些奇怪的bug.
为了解决上面的缺点，可以把模块写成一个对象，所有的模块成员都放到这个对象里面
```javascript
    var module1 = {
        state : '',
        method1 : function(){
            //do somethings
        },
        mehtod2 : function(){
            //do somethings
        }
    }

```
这种写法将方法和变量都封装到了module1里，调用的时候直接 **对象.方法名** 就可以了
```javascript
    module1.mehtod1();
```
但是，这样的写法会暴露所有模块成员，内部变量可以被外部改写。比如，外部的代码可以直接修改state的值。
## 用面向对象构造函数，封装私有对象
利用构造函数的特性，封装私有对象
```javascript
function ModuleClass(){
    var hello = 'hello world';
    this.sayHello = function(){
        console.log(hello);
    };
}

var module = new ModuleClass();
module.sayHello();
```
这样私有变量在外部就不会被访问到，这种把函数直接写在写在构造函数内部的做法，每次实例化的时候都会创建这个函数，非常耗费内存。
利用prototype(原型)改写上面的程序如下:
```javascript
function ModuleClass(){
    this.hello = 'hello world';
}
ModuleClass.prototype.sayHello = function(){
    console.log(this.hello);
}
```
改写之后，将私有变量放入实例对象中，好处是看上去更自然，但是它的私有变量可以从外部读写。
## 利用立即执行函数写法写模块
使用”立即执行函数”（Immediately-Invoked Function Expression，IIFE），将相关的属性和方法封装在一个函数作用域里面，可以达到不暴露私有成员的目的。
```javascript
var module = (function(){
    var hello = 'hello world';
    return {
        sayHello : function(){
            console.log(hello);
        }
    }
})();
```
这样一来，外部就无法访问到私有的hello变量了。
更多的时候我们模块需要一来另一些模块或者全局变量，返回的时候挂载再全局变量上
```javascript
(function(window, $){
    var hello = 'hello world';
    function f1 (){
        //do somethings
    }
    function f2 (){
        //do somethings
    }
    function f3 (){
        //do somethings
    }
    //这里也可以对引入对象进行扩展
    $.fn.test = function(){

    }
    window.module1 = {
        sayHello : function(){
            console.log(hello);
        }
    }
})(window, jQuery);
```
如果这个模块需要让外部调用哪些方法，只用挂载到module1上就可以了,同时，立即执行函数内部也可以对引入对象（例如jQuery，做扩展操作。以保证内部的方法和变量都是独立的，不会影响到外面