---
date: 2015-11-15 00:05
tags: 前端工具 sublime
title: sublime自定义代码提示
---

## snippet
我们在编写代码的时候老是有些代码模板重复使用，常规的做法是copy过来，这种做法几位耗时间，这个时候snippet就来了，
### 1.Snippe创建,存储和格式
Snippet文件是以.sublime-snippet为扩展的XML文件, 可以命名为XXX.sublime-snippet, 创建自己的snippet的方式为菜单栏Tools | New Snippet..
文件格式如果
```xml
<snippet>
    <content><![CDATA[Type your snippet here]]></content>
    <!-- Optional: Tab trigger to activate the snippet -->
    <tabTrigger>hello</tabTrigger>
    <!-- Optional: Scope the tab trigger will be active in -->
    <scope>source.python</scope>
    <!-- Optional: Description to show in the menu -->
    <description>My Fancy Snippet</description>
</snippet>
```
ps:存储在包目录下user目录下即可
简要介绍一下snippet四个组成部分:
+ content:其中必须包含<![CDATA[…]]>,否则无法工作, Type your snippet here用来写你自己的代码片段
+ tabTrigger:用来引发代码片段的字符或者字符串, 比如在以上例子上, 在编辑窗口输入hello然后按下tab就会在编辑器输出Type your snippet here这段代码片段
+ scope: 表示你的代码片段会在那种语言环境下激活, 比如上面代码定义了source.python, 意思是这段代码片段会在python语言环境下激活.（html为 text.html js为source.js）
+ description :展示代码片段的描述, 如果不写的话, 默认使用代码片段的文件名作为描述,会在提示时显示

### 2. snippet环境变量
列举一下可能用到的环境变量, 这些环境变量是在Sublime中已经预定义的.
$TM_FILENAME 用户文件名 
$TM_FILEPATH 用户文件全路径
$TM_FULLNAME	用户的用户名
$TM_LINE_INDEX	插入多少列, 默认为0
$TM_LINE_NUMBER	一个snippet插入多少行
$TM_SOFT_TABS	如果设置translate_tabs_to_spaces : true 则为Yes
$TM_TAB_SIZE	每个Tab包含几个空格
同一通过下面的代码片段进行验证:
```xml
<snippet>
   <content><![CDATA[
=================================
$TM_FILENAME   用户文件名
$TM_FILEPATH   用户文件全路径
$TM_FULLNAME    用户的用户名
$TM_LINE_INDEX   插入多少列, 默认为0
$TM_LINE_NUMBER   一个snippet插入多少行
$TM_SOFT_TABS  如果设置translate_tabs_to_spaces : true 则为Yes
$TM_TAB_SIZE   每个Tab包含几个空格
=================================
]]></content>
    <!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
    <tabTrigger>hello</tabTrigger>
    <!-- Optional: Set a scope to limit where the snippet will trigger -->
    <scope>source.python</scope>
</snippet>
```
验证方式 : 保存自定义snippet,在python文件夹下输入hello按下tab
### 指定光标位置和选中值
```
<snippet>
	<content><![CDATA[
Hello, ${1:this} is a ${2:snippet}.
]]></content>
	<!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
	<tabTrigger>hello</tabTrigger>
	<!-- Optional: Set a scope to limit where the snippet will trigger -->
	<scope>text.html</scope>
</snippet>
```
完成后再html文件中敲下hello然后tab，便可以看到光标选中this的问着在此tab光标选中snippet,1、2代表了选中的顺序如果有两个$1会自动进入多点编辑模式

## Completions
上面的提示方法虽然好但是呢，每个提示都要建一个文件，太麻烦了，如果这是代码方法或者长单词提示这种方法就不太好了，这个时候我们可以定义completions,
文件格式如下：
```json
{
   "scope": "text.html - source - meta.tag, punctuation.definition.tag.begin",

   "completions":
   [
      { "trigger": "hello", "contents": "hello word" },
      {"trigger": "this is", "contents":"this is $1"}
   ]
}
```
存储格式为.sublime-completions
同样的是放在user目录下
参数解释：
scope：作用域与snippet中一致
trigger: 触发词
contents: 内容定义
$ : 这个定义光标位置与snippet一致

## 总结
snippet适用于大段代码提示，completions适用于小段代码提示，都非常好用，快去自定义一套适合自己打代码提示吧！
## 参考链接
+ [手把手教你写Sublime中的Snippet](http://ju.outofmemory.cn/entry/105183)
+ [sublime completions](http://docs.sublimetext.info/en/latest/reference/completions.html)
+ [sublime snippets](http://docs.sublimetext.info/en/latest/extensibility/snippets.html)