---
date: 2016-03-04 12:27
tags: css
title: css布局-盒模型，定位和浮动
---

## 盒模型
简单来说，网页中的额每一个元素都是一个盒子，而整个网页就是有大大小小的盒子组装起来的。既然是盒子那么它就有：内容（content）、填充（padding）、边框（border）、边界（margin）这些属性，而每个属性又有上下左右四个方向，像下面这张图这样

![13-34-30](/images/13-34-30.jpg)

## display
每一个元素都是一个盒子，display属性就决定了他们属于那种盒子；目前大致分为以下几种
### 块级元素（block）
一般是其他元素的容器，可容纳内联元素和其他块状元素，块状元素排斥其他元素与其位于同一行，宽度(width)高度(height)起作用。常见块状元素为div和p。
### 内联元素(inline)
内联元素只能容纳文本或者其他内联元素，它允许其他内联元素与其位于同一行，但宽度(width)高度(height)不起作用。常见内联元素为“a”。
### 内联块级元素(inline block)
同时拥有块级元素和内联元素的特性

## position
display决定了盒子种类，position就确定了盒子的位置，
### （标准文档流）static
常规元素默认值都是static，指按照文档至上而下的标准文档流
### （绝对定位）absolute
绝对定位，他默认参照已经定位(position值不为static)的元素父级元素，配合top、right、bottom、left进行定位。
脱离标准文档流，也就是他的位置不会影响到其他的元素，也不会参与到父元素高度计算上，但是会受到富元素overflow:hidden的影响
### （绝对定位）fixed
参照window配合top、right、bottom、left进行定位。
脱离标准文档流，不会受到父元素overflow:hidden影响;
### （相对定位）relative
相对定位，他是默认参照父级的原始点为原始点，配合top、right、bottom、left进行定位，当父级内有padding等CSS属性时，当前级的原始点则参照父级内容区的原始点进行定位。
## 浮动（float）
首先要知道，div是块级元素，在页面中独占一行，自上而下排列，那么如何让块级元素在一行中显示呢，这里就用到float;
float:left/right可以让块级元素在一行中从左至右或者从右至左排列，但是，需要主要的是，当元素设置了float:left/right时，会影响到后面的元素所以需要及时的清除浮动，而且会造成父级元素高度塌陷。
常用的清除浮动代码如下:
```css
.clear { clear: both; }/*清除浮动*/
/*清除浮动并防止父级元素高度塌陷*/
.clearfix:before, .clearfix:after { 
    content: '\0020'; 
    display: block; 
    overflow: hidden; 
    visibility: hidden; 
    width: 0; height: 0; 
}
.clearfix:after { clear: both }
.clearfix { *zoom: 1 }
```

