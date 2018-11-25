---
date: 2015-11-24 21:43
status: public
title: 前端代码规范(YaHoo军规)
---

1.尽可能减少HTTP请求数（请求耗时）
2.使用CDN（内容分发网络）
3.添加Expire/Cache-Control头（缓存设置）
4.启用Gzip压缩（压缩css和js代码，混淆js代码）
5.把css放在页面最上面（方便加载渲染）
6.把script放在页面最下面（防止script代码影响页面展示）
7.避免在css中使用Expression
8.把JavaScript和css放在外部分文件中（js和css文件浏览器是自动缓存的不会重复加载）
9.减少DNS查询
10.避免重定向
