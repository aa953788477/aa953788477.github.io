---
title: css布局-BFC和IFC
date: 2016-03-04 12:29
tags: css
---

Formatting Contexts是W3C CSS2.1规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。
## BFC
BFC(Block Formatting Contexts)直译为"块级格式化上下文"。Block Formatting Contexts就是页面上的一个隔离的渲染区域，容器里面的子元素不会在布局上影响到外面的元素,反之也是如此。
### BFC形成

> * float 的值不为 none
> * position 的值不为 static 或 relative
> * display属性为inline-boxs、table-cells、table-captions的不是块盒的块容器(除非这个值已经被传播到视口),
> * overflow不为visible的块盒才会为它的内容创建新的BFC。
> * body元素

### BFC渲染规则

> 1. 内部的Box会在垂直方向，一个接一个地放置。
> 2. Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠
> 3. 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
> 4. BFC的区域不会与float box重叠，常用来清除浮动和布局。
> 5. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
> 6. 计算BFC的高度时，浮动元素也参与计算

## IFC
IFC(Inline Formatting Contexts)直译为"内联格式化上下文"，IFC的line box（线框）高度由其包含行内元素中最高的实际高度计算而来（不受到竖直方向的padding/margin影响)
### IFC渲染规则

> 1. 框会从包含块的顶部开始，一个接一个地水平摆放。
> 2. 摆放这些框的时候，它们在水平方向上的外边距、边框、内边距所占用的空间都会被考虑在内。在垂直方向上，这些框可能会以不同形式来对齐：它们可能会把底部或顶部对齐，也可能把其内部的文本基线对齐。能把在一行上的框都完全包含进去的一个矩形区域，被称为该行的行框。水平的margin、padding、border有效，垂直无效。不能指定宽高。
> 3. 行框的宽度是由包含块和存在的浮动来决定。

### IFC  ‘line-height’ 与 ‘vertical-align’ 属性

> 1. 计算行框里的各行内级框的高度。对于置换元素、行内块元素、行内表格元素来说，这是边界框的高度，对于行内框来说，这是其 ‘line-height’。
> 2. 行内级元素根据其 ‘vertical-align’ 属性垂直对齐。在这些框使用 ‘top’ 或 ‘bottom’ 对齐的情况下，user-agent必须以最小化行框的高为目标对齐这些框。若这些框够高，则存在多个解而 CSS 2.1 不定义行框基线的位置。

> 3. 行框的高是最顶端框的顶边到最底端框的底边的距离。