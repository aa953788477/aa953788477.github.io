---
title: javascript类型判断
date: 2015-11-09 17:26
tags: javascript
---

## 判断String类型
```javascript
function isString(value) {
	return typeof value === 'string';
};
```
## 判断Number类型
```javascript
function(value) {
	return typeof value === 'number';
};
```
## 判断function
```javascript
function isFunction(value) {
	return typeof value === 'function';
};
```
## 判断正则表达式类型
```javascript
function isRegExp(value) {
	return toString.call(value) === '[object RegExp]';
}
```
## 判断是否是对象
```javascript
function isObject(value){
    return value !== null && typeof value === 'object';
}
```
## 判断是否是Object的对象
```javascript
function isBlankObject(value) {
		return value !== null && typeof value === 'object' && !     getPrototypeOf(value);
};
```
## 判断是否是数组
```javascript
function isArray(value){
    if (Object.prototype.toString.apply(arr) === '[object Array]') 
        return true;
    else 
        return false;
}
```