---
title: Flex 布局笔记
date: 2018-03-06 12:44:35
tags:
---

一 、Flex 布局是什么？

* flex 是 Flexible Box 的缩写，意为"弹性布局"
* 任何一个容器都可以指定为 Flex 布局：
```css
.box{
  display: flex;
}

/* 行内元素 */
.box{
  display: inline-flex;
}
```
* 注意，设为 Flex 布局以后，子元素的float、clear和vertical-align属性将失效。

二、基本概念

* 采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。
* 容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。
* 默认是从左上角到右下角，还是 normal flow
* 项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size。

三、容器的属性
* flex-direction
  - 决定主轴的方向（即项目的排列方向）
  - flex-direction: row | row-reverse | column | column-reverse;
* flex-wrap
  - 如果当主轴尺寸固定时，当空间不足时，如何换行
  - flex-wrap: nowrap | wrap | wrap-reverse;
  - wrap-reverse 会将 main start 定位下方
* flex-flow
  - 是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap
* justify-content
  - 定义项目在主轴上的对齐方式
  - 不会改变 main start 的位置
  - justify-content: flex-start | flex-end | center | space-between | space-around;
* align-items
  - 定义项目在交叉轴上如何对齐
  - align-items: flex-start | flex-end | center | baseline | stretch;
  - 只有一排项目的时候才能生效
* align-content
  - 定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用
  - align-content: flex-start | flex-end | center | space-between | space-around | stretch;

四、项目的属性
* order
  - 定义项目的排列顺序。数值越小，排列越靠前，默认为0
* flex-grow
  - 定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大
  - 如果所有项目的flex-grow属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的flex-grow属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。
* flex-shrink
  - 定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小
* flex-basis
  - 定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。
  - 当主轴为水平方向的时候，当设置了 flex-basis，项目的宽度设置值会失效，flex-basis 需要跟 flex-grow 和 flex-shrink 配合使用才能发挥效果。
* flex
  - 是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选
  - 快捷值：
    * auto (1 1 auto)
    * none (0 0 auto)
    * flex: 1;（num, 1 0%）
* align-self
  - align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。
  - align-self: auto | flex-start | flex-end | center | baseline | stretch;