---
date: 2016-08-16 08:25
title: ES6的二三事(一)
tags: 
- javascript 
- ES6
---

最近都在开发 `react-native` 和 `Vue` 的项目，接触多了 `ES6` 代码，相较于之前的一脸懵逼，显示觉得 `ES6` 简直妙不可言，于是赶紧写写博客总结下知识点和经验。

## 简介
>ECMAScript 6.0（以下简称ES6）是JavaScript语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得JavaScript语言可以用来编写复杂的大型应用程序，成为企业级开发语言。
>
>标准的制定者有计划，以后每年发布一次标准，使用年份作为版本。因为ES6的第一个版本是在2015年发布的，所以又称ECMAScript 2015（简称ES2015）。

<!--more-->

## let 和 const 命令
在之前的 javascript 开发中，我们使用 `var` 定义变量，这种方式定义的变量有几个问题:

> 1. 没有块级作用域
> 2. 存在变量提升
> 3. 没有常量的定义

为了解决这些问题， ES6 新增了 `let` 命令来定义变量， `const` 命令来定义常量, 如果最 javascript 作用域不太了解的，可以看我之前的博客 [javascript作用域和闭包](http://www.myronliu.com/2016/03/30/javscript/javascript%E4%BD%9C%E7%94%A8%E5%9F%9F%E5%92%8C%E9%97%AD%E5%8C%85/)

### 块级作用域

javascript 之前用 `var` 定义的变量是没有块级作用域的，在 `if` 或 `for` 语句块中定义的变量，在外面仍可以访问到.

```javascript
if (true) {
    var test = 'this is test msg'
}
console.log(test) // this is test msg

for (var i = 0; i < 5; i++) {
}
console.log(i)  // 5
```

是用 `let` 定义变量之后，就可以有块级作用域了.

```javascript
for (let i = 0; i < arr.length; i++) {}

console.log(i);
//ReferenceError: i is not defined
```

### 不存在变量提升

```javascript
console.log(foo); // 输出undefined
console.log(bar); // 报错ReferenceError

var foo = 2;
let bar = 2;
```

>上面代码中，变量 `foo` 用 `var` 命令声明，会发生变量提升，即脚本开始运行时，变量 `foo` 已经存在了，但是没有值，所以会输出undefined。变量 `bar` 用 `let` 命令声明，不会发生变量提升。这表示在声明它之前，变量 `bar` 是不存在的，这时如果用到它，就会抛出一个错误。

### 常量

ES6之前，没有常量的定义，只能像这样写

```javascript
var SIZE = 10;  // 用大写定义变量约定为常量，后面不可修改
```

以上的做法还是会有后面被人修改的隐患， 于是ES6中有了 `const` 定义常量

```javascript
const SIZE = 10;

SIZE = 4; // 报错
```

## 变量的结构赋值

> ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

以前，为变量赋值，只能直接指定值。

```javascript
var a = 1;
var b = 2;
var c = 3;
```

ES6 中允许可以写成这样

```javascript
var [a, b, c] = [1, 2, 3];
```

上面是在数组提取值，除此之外，ES6还支持对象中提取值

```javascript
var {a, b, c} = {a:1, b: 2, c: 3}
// a = 1
// b = 2
// c = 3
```

## 箭头函数

ES6允许使用“箭头”（=>）定义函数。

```javascript
var fun = v => v;

// 等同于

var fun = function (v) {
    return v;
}
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```javascript
var f = () => 5;
// 等同于
var f = function () { return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};
```

另外用 `=>` 定义的函数，是绑定了 *this* 对象的

```javascript
var f = () => 5

// 等同于

var f = function () {
    return 5;
}.bind(this)

```

除此之外箭头函数使用还需要注意以下三点：

> 1. 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
> 2. 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替。
> 3. 不可以使用yield命令，因此箭头函数不能用作Generator函数。

## class

JavaScript语言的传统方法是通过构造函数，定义并生成新对象

```javascript
function Person (name, age) {
    this.name = name;
    this.age = age;
}

// 利用原型绑定方法，不会每次实例化都创建，提升性能
Person.prototype.say = function () {
    console.log('My name is ' + this.name + ', I`m ' + this.age + ' years old');
}

var hong = new Person('小红', 16)
```

这种定义的方式有存在以下问题：

> 1. 和 java、c# 等传统面向对象语言，定义方式差别很大，容易让然产生困惑
> 2. 没有继承机制，只能通过原型链模拟，需要自己实现
> 


ES6提供了更接近传统语言的写法，引入了Class（类）这个概念，作为对象的模板。通过class (class在之前的版本一直是保留关键字)关键字，可以定义类。

```javascript
class Person {
    constructor (name, age) {
        this.name = name;
        this.age = age;
    }
    say () {
        console.log('My name is ' + this.name + ', I`m ' + this.age + ' years old');
    }
}
```

使用方式跟之前一样

```javascript
var hong = new Person('小红', 16)
```

可以通过 `extends` 关键字继承类

```javascript
class Man extends Person {
    constructor (name, age) {
        super(name, age) // 通过 super 调用父类的同名方法
    }
}
```

`static` 关键字定义静态属性和方法


```javascript
class Person {
    constructor (name, age) {
        this.name = name;
        this.age = age;
    }
    say () {
        console.log('My name is ' + this.name + ', I`m ' + this.age + ' years old');
    }
    static test = '111'
    static desc () {
        console.log('this is person class');
    }
}

console.log(Person.test) // 111
Person.desc()

var hong = new Person('小红', 16) 
hong.desc() // 报错，没有这个方法
```

> 基本上，ES6的class可以看作只是一个语法糖，它的绝大部分功能，ES5都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。


