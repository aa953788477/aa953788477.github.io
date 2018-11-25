---
date: 2016-03-30 12:24
title: javascript作用域与闭包
tags: javascript
---

## 作用域
在编程语言中，作用域控制着变量的与参数的可见性和生命周期。在javascript中并不像其他编程语言一样拥有块级作用域，在if中定义的变量再外面也可以直接调用,这点是值得注意的;
```javascript
var a= 1;
if(true){
    var a = 2;
}
console.log(a); //2
```
在块中定义的变量，再外边也可以直接调用到，也可以覆盖该作用域内同名的变量;

<!--more-->

javascript中最外层是再window(浏览器环境中)下的全局作用域；每个函数内部都会产生一个函数作用域，我们在访问一个变量的时候，会本着就近原则，从所在的函数作用域逐层向上查找，如果没有找到，就会抛出undefined异常
```javascript
var a = 'window';
function test(){
    var a = 'func';
    console.log('this test a:' +a);
}
test();
console.log('this window a:' + a);
// this test a:func
// this test a:window
```

**关于变量提升**
```javascript
var a = 'window'；
function test(){
    console.log('this test a:' +a);
    var a = 'func';
}
test();
console.log('this window a:' + a);
// this test a:undefined
// this test a:window
```
很奇怪吧，再方法中输出a的时候，应该是全局变量中的a变量才对,javascript执行函数式，会先创建变量，然后执行语句赋值，所以在执行 **console.log** 之前已经定义了变量 **a** 但是并未赋值，这个时候 **a=undefined**,然后执行 **console.log**自然就输出undefined；

## 闭包
作用域的好处是内部函数可以访问和定义他们的外部函数的参数和变量，而函数外部是无法直接访问无法直接访问到函数内部的变量的。那么我们怎么访问到函数内部的变量呢, 通常在函数内部再创建一个函数，再返回这个函数再外部调用;
```javascript
function f1(){
    var test = '这是个私有变量';
    function f2(){
        console.log(test);
    }
    return f2;
}
var result = f1();
result(); // “这是个私有变量”=
```
这里的f2就是闭包，我对闭包的定义呢，**闭包就是能够读取到函数内部变量的函数；**

在上面的代码中我们看到使用闭包可以访问都函数内部的变量，再实际运用中闭包还有一个好处就是可以使得函数内部变量常驻在内存中;
```javascript
function f1(){
    var count = 1;
    return function (){
        return count++;
    }
}
var count = f1();
console.log(count()); //1
console.log(count()); //2
console.log(count()); //3
console.log(count()); //4
```

我们在封装一些类库的时候，需要有一些私有变量不给外部直接调用，这个时候就需要使用到闭包；
```javascript
//缓存信息的库
var cache = function(){
    var map = {};
    return {
        get : function(key){
            return map[key];
        },
        set : function(key, value){
            map[key] = value;
        },
        remove : function(key){
            if(map[key]) delete map[key];
        }
    };
}
cache.set('test', 'this is test case');
cache.get('test'); //this is test case
cache.remove('test');
```
这里我们封装了一个简单cache对象用于缓存一些信息， 使用内部变量 **map** 存储，外部无法直接访问 **map** 变量，但是可以通过 **set** **get** **remove** 完成对 **map** 增删改查的操作;

使用闭包需要注意的：

> 1. 闭包会是函数作用域中所有的变量常驻在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
> 2. 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（ **object** ）使用，把闭包当作它的公用方法（ **Public Method**），把内部变量当作它的私有属性（**private value**），这时一定要小心，不要随便改变父函数内部变量的值。

```javascript
function f1(){
    var TEST = '11111'//我是一个常亮，不要改变;
    var nouse = 'aaa'; //这是一个没有使用到的变量再f1执行完成前要释放掉
    function f2(){
        //通过TEST做一些操作
        console.log(TEST);//非要条件，不要轻易改变父函数内部变量的值；
    }
    nouse = null; //要吧这些不用的释放掉
    return f2;
}
```

## this
每一个函数内部都有两个固定的变量 **this** 和 **arguments**, **arguments** 是调用函数是传入参数的集合; **this** 指向调用函数的对象
```javascript
var  test = {
    fun : function (){
        console.log(this, arguments);
    }
}
test.fun(); // test, []
test.fun(1,2); // test, [1,2]
test.fun(1,2,3); // test, [1,2,3]
```

**函数中的this 永远指向调用它的对象,如果没有指定对象调用，那么this指向全局对象**
```
var  test = {
    fun : function (func){
        console.log(this === test, arguments);
        func();
    }
}
test.fun(function (){
    console.log(this === window);
});
//true
//true
```
上面的代码中 **test** 中的 **fun** 函数是通过 **test** 对象调用的，所以 **fun** 中的 **this** 指向test, 而传入到fun中的方法 **func** 并没有通过 **对象名.方法名** 调用,所以是使用的默认的全局对象（ **window** ）调用的 **func** 方法。

## call、apply和bind
在javascript中调用一个方法，除了常用的 **对象名.方法名** 的调用方式外，还可以种用 **Function** 对象自带的 **call** 和 **apply** 方法调用, 调用时需要指定 用哪个对象调用和传入参数
```javascript
var test = {
    msg : 'test msg'
};
var msg = 'window msg';
function say(){
    console.log(this.msg);
}
//没有指定对象 默认就是window
say.call();         // window msg
say.call(null);     // window msg
say.call(this);     // window msg

say.call(test);     // test msg
```

**apply** 方法和 **call** 方法都可以动态执行this的指向对象，只是在调用上有区别

```javascript
apply(obj, [arg1, arg2, arg3]);
call(obj, arg1, arg2, arg3);
```

**bind** 方法在 **function** 定义的时候使用，给 **function** 指定 **this** 指向, 当一个函数使用 **bind** 方法指定了 this指向后，那么在调用的时候 **call** 和 **apply** 执行的 **this** 指向都将无效

```javascript
var test = {
    msg :'test msg',
    fun: function(){
        console.log(this.msg);
    }.bind(window); //强制指定 window 为 this;
};
var msg = 'window msg';
test.fun(); // window.msg
test.fun.call(test); // window.msg
test.fun.apply(test); // window.msg
```

**bind** 方法是 ES5中新增的，ie8 上不支持，我们可以自定义bind来hack

```javascript
if(!Function.prototype.bind){
    Function.prototype.bind = function(context){
        var _self = this;
        return function(){
            return _self.apply(context, arguments)
        };
    }
}
```
