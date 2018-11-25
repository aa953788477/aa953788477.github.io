---
date: 2016-03-23 19:19
tags: javascript
title: javascript面向对象编程
---

“面向对象编程”（Object Oriented Programming，缩写为OOP）是目前主流的编程范式。它的核心思想是将真实世界中各种复杂的关系，抽象为一个个对象，然后由对象之间的分工与合作，完成对真实世界的模拟。

## 对象的理解
那么，什么是对象呢？我们可以从两个层次理解：

*（1）“对象”是单个实物的抽象。*

一本书、一辆汽车、一个人都可以是“对象”，一个数据库、一张网页、一个与远程服务器的连接也可以是“对象”。当实物被抽象成“对象”，实物之间的关系就变成了“对象”之间的关系，从而就可以模拟现实情况，针对“对象”进行编程。

*（2）“对象”是一个容器，封装了“属性”（property）和“方法”（method）。*

所谓“属性”，就是对象的状态；所谓“方法”，就是对象的行为（完成某种任务）。比如，我们可以把动物抽象为animal对象，“属性”记录具体是那一种动物，“方法”表示动物的某种行为（奔跑、捕猎、休息等等）。
<!--more-->
## 构造函数
“面向对象编程”的第一步，就是要生成“对象”。前面说过，“对象”是单个实物的抽象。通常需要一个模板，表示某一类实物的共同特征，然后“对象”根据这个模板生成。

典型的面向对象语言（如：c#/java）,都有 *类* 这个概念用于构造对象， 对象就是类的实例，类就是对象的模板，但是，JavaScript语言的对象体系，不是基于“类”的，而是基于构造函数（constructor）和原型链（prototype）。

> 在ES6中也有了类的概念来构造对象

构造函数是一个正常的函数，但是它的特征和用法与普通函数不一样。下面就是一个构造函数。

```javascript
function Person(){
    this.name = 'myron';
    this.age = 22;
}
```
可以看到，Person就是构造函数，通过this属性来封装自身的属性和方法，构造函数的特点有两个：

> 1. 函数内部使用了this关键字，指向生成的对象;
> 2. 必须用 *new* 命令调用函数生成对象；

```javascript
var myron = new Person();
```
上面的代码就创建了一个新的对象，而new命令的执行过程是先创建一个空的对象，然后用这个对象去掉用构造函数完成是实例化操作，执行构造函数时会忽略到构造函数中的return语句,下面的代码来模拟new命令的执行过程;

```javascript
function _new(/*构造函数，实例化参数*/){
    var args = [].slice.call(arguments);
    var constructor = args.shift();
    var context = Object.create(constructor.prototype);//根据原型链创建新的对象
    var result = constructor.apply(context, args); //调用构造方法
    return result;
}
```

## 原型
JavaScript通过构造函数生成新对象，因此构造函数可以视为对象的模板。实例对象的属性和方法，可以定义在构造函数内部。
```javascript
function Person(name, age){
    this.name = name;
    this.age = age;
    this.introduce = function(){
        console.log('my name is ' + this.name + ', ' + this.age + ' years old' );
    }
}
var myron = new Person('myron', 22);
var ling = new Person('若绫', 20);

console.log(myron.introduce === ling.introduce);//false
```
在上面的代码中创建了两个实例, 但是它们的 *introduce* 方法却不一样，也就是，我们写在构造函数中的函数和属性在每次实例化得时候都会重新创建。对于一些公用的资源（函数/属性）来说，这既没有必要，又浪费系统资源，因为这些完全可以共享。

我们创建的每个函数都有一个 prototype(原型)属性,这个属性是一个对象,它的用途是
包含可以由特定类型的所有实例共享的属性和方法。逻辑上可以这么理解: prototype 通过 调用构造函数而创建的那个对象的原型对象 。使用原型的好处可以让所有对象实例共享它所 包含的属性和方法。也就是说,不必在构造函数中定义对象信息 ,而是可以直接将这些信息 添加到原型中。
```javascript
function Person(name, age){
    this.name = name;
    this.age = age;
}
Person.prototype.introduce = function(){
    console.log('my name is ' + this.name + ', ' + this.age + ' years old' );
}
var myron = new Person('myron', 22);
var ling = new Person('若绫', 20);
console.log(myron.introduce === ling.introduce);//true
```
为了更进一步了解构造函数的声明方式和原型模式的声明方式 ,我们通过图示来了解一下:

![19-21-26](/images/19-21-26.jpg)

![19-21-43](/images/19-21-43.jpg)

在原型模式声明中,多了两个属性,这两个属性都是创建对象时自动生成的。__proto__ 属性是实例指向原型对象的一个指针,它的作用就是指向构造函数的原型属性 constructor。 通过这两个属性,就可以访问到原型里的属性和方法了。

原型模式的执行流程:
1.先查找构造函数实例里的属性或方法,如果有,立刻返回;
2.如果构造函数实例里没有,则去它的原型对象里找,如果有,就返回;

## 继承
javascript没有 *类* 的概念，自然也没有类的继承(ES6除外),我们只能通过一些独特的方法完成继承操作.

### 属性拷贝
这是最简单也最容易理解的继承方法，便利一个对象的所有属性，然后设置给另一个对象
```javascript
var a = {
    a: 'aa'
}
var b = {
    b: 'bb'
}
for(var key in a){
    b[key] = a[key]
}
```
上述继承方式只针对Object继承，只是对单个对象有用，又有一个对象需要继承a则需要重新遍历一次,所以我们函数再构造函数上动手脚吧

### 构造继承
再子类构造函数中用this对象调用父类构造函数方法，完成子类对父类的继承关系
```javascript
function Animal(){
    this.age = 10;
    this.weight = '50kg';
}

function Cat(){
    Animal.call(this);
}
var cat = new Cat();
console.log(cat.age === 10 ? '完成继承啦' : '');
```
但是这种方法，只能继承父类中构造函数定义的属性和方法，如果函数定义在原型中则无法继承;

### 原型继承
之前就说每个函数都有prototype属性，用于共享属性和方法，但同同时它本身就是一个对象，也有自己的prototype,这样一来就形成一条原型继承链。
```javascript
function Animal(){
    this.age = 10;
    this.weight = '50kg';
}
function Cat(name){
    this.name  = name; //定义自己的属性
}
Cat.prototype = new Animal();//继承animal属性和方法
Cat.prototype.say = function(){  //新增一些原型属性和方法
    console.log('喵~');
}
```
利用原型继承可以完成做父类所有的方法和属性的继承，也可以定义一些子类特有的属性和方法.
推荐一下jQuery作者写的类工厂的实现 [点这里](http://ejohn.org/blog/simple-javascript-inheritance/)

它通过 *Class.create* 方法创建类，需要继承则调用 *extend* 函数，另外在函数内部this._super指向父类的同名方法.
```javascript
var Person = Class.extend({
  init: function(isDancing){
    this.dancing = isDancing;
  },
  dance: function(){
    return this.dancing;
  }
});

var Ninja = Person.extend({
  init: function(){
    this._super( false );
  },
  dance: function(){
    // Call the inherited version of dance()
    return this._super();
  },
  swingSword: function(){
    return true;
  }
});

var p = new Person(true);
p.dance(); // => true

var n = new Ninja();
n.dance(); // => false
n.swingSword(); // => true

// Should all be true
p instanceof Person && p instanceof Class &&
n instanceof Ninja && n instanceof Person && n instanceof Class
```

## 混合
Mixin是JavaScript中用的最普遍的模式，几乎所有流行类库都会有Mixin的实现。
Mixin是掺合，混合，糅合的意思，即可以就任意一个对象的全部或部分属性拷贝到另一个对象/类上。
```javascript
//基本的混合实现
var mixin = {
    say : function(){
        console.log(this.sayMsg);
    }
};
function Cat(name){
    this.name  = name; //定义自己的属性
    this.sayMsg = '喵~';
}
function Dog(name){
    this.name = name;
    this.sayMsg = '汪!汪!汪!';
}
//把mixin加入到两个类中
for(var key in mixin){
    Cat.prototype[key] = mixin[key];
    Dog.prototype[key] = mixin[key];
}
var cat = new Cat('小黄');
var dog = new Dog('旺财');
cat.say();
dog.say();
```
很多时候，我们只需要使用到某个类中的一个或者两个方法，来避免重复定义函数，这个时候如果用继承来实现，那么代价就太大了，而且在多次继承之后，子类方法很可能写会不经意覆盖掉了祖类（父类的父类。。。）的方法导致对象有些操作不能正常完成。

因此我们只需要两级或者三级继承，将一些公用的功能或是函数都抽出来，作为一个个独立的object，需要使用到这些，就利用混合的方法加入到类中。这种方式，可以极大的提升代码的复用性。
我自己的实现：[可以继承和混合的类工厂](https://github.com/aa953788477/itools/blob/master/src/class/class.js)


