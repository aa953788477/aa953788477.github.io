---
title: HTML5 语义化
date: 2016-05-29 22:31:34
tags:
- webapp
- 移动端
- HTML5
---


## 以往的开发方式

```html
 <div class="">
     <div class="">
         <div class="">
             .....
         </div>
     </div>
 </div>
```

我们在使用HTML4.1或是XHTML的开发的时候，基本上每个页面上都只有 `div` 和 `span` 标签， 浏览器并不知道每一个元素属于页面的那个部分。

<!--more-->

## 为什么要语义化

在W3C标准中，行为、表写、结构要分离，而HTML则负责网页的结构，更为语义化的标签可以让代码的结构更为清晰

```html
<header>
    <nav>

    </nav>
</header>
<section>
    <article class="">
        <header>

        </header>
        <section>

        </section>
        <footer>
        </footer>
    </article>
</section>
<footer>
</footer>
```

这样写出的代码可以一眼看出每一块代表的什么意思，当然语义化的好处还不只这些

### 1.有利于SEO优化

页面上元素更为清晰，当小蜘蛛（搜索引擎爬数据的）访问页面的时候，可以更加快捷清楚的抓取到当前的网页的重要信息，更为语义化的代码可以是你的网站在搜索引擎中排名靠前。

### 2.无障碍阅读

如果访客有视障，那么他只能通过屏幕阅读器来讲网页的信息读取出来播放给访客，更为语义化的代码可以让屏幕阅读器更容易理解。

### 3.有些设备可能性能比较荣，想pc上浏览器一样渲染页面

这些设备对于css的支持比较弱，语义标记为设备提供了所需的相关信息，就省去了你自己去考虑所有可能的显示情况（包括现有的或者将来新的设备）。例如，一部手机可以选择使一段标记了标题的文字以粗体显示，而掌上电脑可能会以比较大的字体来显示，无论哪种方式一旦你对文本标记为标题，您就可以确信读取设备将根据其自身的条件来合适地显示页面。

## HTML5的语义化标签

HTML5新增了许多的语义化标签，像之前代码中有用到的 `header` `footer` `nav`等等

### header

用于描述定义每一块或者页面的头信息（页眉）;

### footer

用于描述定义每一块或者页面的页脚信息；

### nav

定义导航链接。

### aside

定义页面内容之外的内容,侧边栏等等.

### article

定义文章。

### section

定义某个部分。

### abbr

定义缩写。

### cite

定义小段引用

### dfn

被定义的词汇 `<dfn>organic food</dfn>`， 表明这一个关键词

### code

定义代码块

### kbd

表示提示键盘输入

### samp

表示计算机输出。

### time

`<time datetime="2011-11-18">November 18th</time>` 标记一个时间，让计算机识别

### mark

定义有记号的文本。

###  wbr

用于定义长单词的换行点，当一行字超出容器范围时，定义这个标签的单词会从这里折到第二行

### q

定义引用，会默认在前后加上引号

### 更加语义的表单

```html
<input type="number" name="name" value=""> <!--数字框-->
<input type="email" name="name" value="">  <!--邮件地址-->
<input type="date" name="name" value="">   <!--日期-->
<input type="datetime" name="name" value=""> <!--时间-->
<input type="range" name="name" value="">    <!--范围-->
```

新增的表单在mobile端效果更为明显，例如 如果设置为 `number` 或 `email` 输入时会弹出相应的键盘。
