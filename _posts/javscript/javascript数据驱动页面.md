---
date: 2016-03-29 15:21
title: javascript数据驱动页面
tags: javascript
---

2005 *Ajax* 提出，web2.0的时代到来，web开发开始越来越注重与用户交互，于是乎，越来越多的后端逻辑迁移到前端实现，javascript代码量大大提升，但是，大量的业务逻辑也ui逻辑混在一起，导致程序臃肿，代码难以维护。于是开始有了，一些类似 *spring* 的分层的思想，将业务逻辑和ui逻辑分开。
<!--more-->
## MVC( [backbone](https://github.com/jashkenas/backbone) ）
MVC是一个架构设计模式，它通过分离关注点的方式来支持改进应用组织方式。它促成了业务数据(Model)从用户界面(View)中分离出来，还有第三个组成部分(Controller)负责管理传统意义上的业务逻辑和用户输入。

![](http://image.beekka.com/blog/2015/bg2015020105.png)

> Model 数据模型一般以json对象的形式展现
> View表示表现层，也就是用户界面，对于网页来说，就是用户看到的网页HTML代码。
> Controller 表示控制层，用来对原始数据（Model）进行加工，传送到View。

mvc结构中 view是根据 *模板* + *数据（Model）* 生成的一段html代码，当我们的用户进行点击按钮或者输入等等操作的时候，就会产生一个动作（ *action* ）view会把 *action* 传递给controller，controller中进行逻辑处理对原始model进行修改，view监控到model的改变之后，通过模板和新的数据重新渲染页面。

在项目中往往采用更加灵活的方式，以 [backbone](https://github.com/jashkenas/backbone) 为例：

![](http://image.beekka.com/blog/2015/bg2015020108.png)

用户可以向 *View* 发送指令（*DOM* 事件），再由 *View* 直接要求 *Model* 改变状态

用户也可以直接向 *Controller* 发送指令（改变 *URL* 触发 *hashChange* 事件），再由 *Controller* 发送给 *View*。

*Controller* 非常薄，只起到路由的作用，而 View 非常厚，业务逻辑都部署在 *View*。所以，*Backbone* 索性取消了 *Controller*，只保留一个 *Router（路由器）* 。

## MVP（[React](http://reactjs.cn/)）
**MVP** 是基于 **MVC** 模式一种演变， P代表的是 Presenter（展示器），其模式如下图所示：

![](http://image.beekka.com/blog/2015/bg2015020109.png)

我们可以看到 view和model是通过展示器做联系，当view中发生了动作（Dom事件)会先传递给展示器，由展示进行逻辑处理，改变model，同时展示其还监控了model的变化，当model发生改变的时候展示其会做相应的逻辑处理并重新渲染页面。

这种做法 **View** 非常薄，不部署任何业务逻辑，称为"被动视图"（ **Passive View** ），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署在那里。

实际使用中以 **react** 为例:
在react中 以 **virtual dom** 就是展示器，而组件的状态 就是model, 当页面上产生事件的时候，会先传递到 **virtual dom** dom触发组件中的事件，组件中进行逻辑处理，改变调用 **setState** 方法改变状态， 组件会重新监控状态改变，调用 **render** 方法产生新的 **virtual dom**，再通过 **virtual dom** 的对比，得出哪些节点发生改变，然后更新改变的节点的真实dom;

## MVVM( [angular](http://www.ngnice.com/) [angular2](https://angular.io/) [vue](http://cn.vuejs.org/))
MVVM 模式将 Presenter 改名为 ViewModel，基本上与 MVP 模式完全一致。

![](http://image.beekka.com/blog/2015/bg2015020110.png)

唯一的区别是，它采用双向绑定（data-binding）; viewModel会观察 model的变化，当model发生改变的时候，viewModel会改变绑定了这个 model 的dom，同时 viewModel也会监控view的变化，当view发生改变的时候，其绑定的model也会发生改变

angular,vue等框架都是基于mvvm模式的一种实现

