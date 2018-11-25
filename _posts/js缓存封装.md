---
title: js缓存封装
date: 2016-05-01 22:44:56
tags:
- javascript
- 缓存
- web storage
---

最近写了许多webapp应用，webapp与传统网页最大的不同在于对于缓存的处理，我们手机端的网络环境较差，如果合理使用缓存可以减少网络请求，加快页面载入速度。
<!--more-->
## web storage

**web storage** 是html5 api中的重要特性，用于缓存数据，与传统的cookie缓存优势明显：

> 1. 容量较大，每个浏览器不同理论值在 4M 以上
> 2. 不会再请求时来回传输。
> 3. 有现成的操作API（setItem/getItem ...）,不用自己封装处理

**web storage** 目前ie8以上浏览器都提供支持；主要分为 **localStorage** 和 **sessionStorage**
> * sessionStorage 缓存在回话中，当浏览器关闭，则缓存失效;
> * localStorage 缓存在本地，只要没有被清除，则一直生效;

**webstorage 的api**

> * setItem (key, value)  缓存数据 key value 格式
> * getItem (key)         获取缓存数据
> * removeItem (key)      删除缓存数据

webstorage 虽然好用，但是仍然不能满足我们现在的需求，主要体现如下：

> 1. api过于简单， 需要扩展一些更全面的api
> 2. 无法缓存Object, 犹豫是以文本方式缓存所以只能处理字符串
> 3. 无法设置缓存时长，例如我只想缓存某个数据10分钟

接下来就是对于 **webstorage** 的扩展封装

## 缓存object问题处理

之前说过，webstorage是以文本方式缓存数据，所以我们先要把Object 序列化问字符串， 这自然就想到了 **JSON**

```javascript
//这里都用localStorage做示例
var store = {}; //封装成store对象操作

store.set = function(key, value){
    var item = {
        data: value
    };
    localStorage.setItem(key, JSON.stringify(item));
}

store.get = function(key){
    var item = localStorage.getItem(key);
    if(!item) return null;
    item = JSON.parse(item);
    return item;
}
```

不是 **Object** 类型的数据不需要序列化，为了统一处理，所以我们创建一个新的object包装数据，提供一致的序列化处理；

## 缓存时长的处理

**webstorage** 并不能像cookie一样设置缓存事件，所以我们需要自己一些手段hack; 步骤如下：

> 1. 用当前事件 + 设置的缓存时长 得到 缓存数据的最后有效时间;
> 2. 在包装对象中加入 end 属性为缓存最后的有效时间；
> 3. 获取缓存数据时根据当前时间和缓存最后有效时间比较，如果过期则删除缓存返回null;

```javascript
store.set = function(key, value, time){
    var item = {
        data: value,
        end: new Date().getTime() + time
    };
    localStorage.setItem(key, JSON.stringify(item));
}

store.get = function(key){
    var item = localStorage.getItem(key);
    if(!item) return null;
    item = JSON.parse(item);
    if(new Date().getTime() > item.end){
        item = null;
        localStorage.removeItem(key);
    }
    return item;
}
```

## 更多的api

直接上代码

```javascript
//获取缓存长度
storage.size = function(){
    return localStorage.length;
}
//遍历缓存
storage.each = function(func){
    for(var key in localStorage){
        if(func(store.get(key), key) === false) return;
    }
}
//多个数据同时缓存
storage.setAll = function(obj){
    for(var key in obj){
        storage.set(key, obj[key])
    })
}
//清除所有的k缓存
storage.clear = function(){
    storage.each(function(item, key){
        localStorage.removeItem(key)
    })
}

//...... 剩下的根据自己的需要补全就好
```

一个相对完善的缓存代码封装就完成。还是比较容易的，敢于尝试，编码就越来越开心。
