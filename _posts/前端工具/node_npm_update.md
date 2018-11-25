---
date: 2016-08-12 14:00
title: node 和 npm 版本升级
tags: 
- 前端工具 
---

今天看 [vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0) 发现必须先把 node 版本升级 6.0以上, 不想再重新下载一次node，于是找到了一个超简单的方法升级 node 和 npm。

<!--more-->

## node 升级

首先安装 n 模块， 它是专门管理node 版本的

```shell
npm install -g n
```

然后，升级node到最新的稳定版

```shell
n stable
```

没错就是这么简单, 后面也可以直接跟版本号哦

```shell
n 6.3.1
```

## npm 升级

npm 无需安装任何东西就可以完成升级

```shell
npm install -g npm
```


