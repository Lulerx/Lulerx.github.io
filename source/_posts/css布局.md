---
title: CSS布局
date: 2020-09-3
categories: [前端,CSS]    #分类
tags: [前端,CSS布局]          #标签 &nbsp;
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ../
---

## 引言

&nbsp;&nbsp;&nbsp;&nbsp; 在我们写网页是，如果只想把所有内容都塞进一栏里，那么不用设置任何布局也是OK的。然而，如果用户把浏览器窗口调整的很大，这时阅读网页会非常难受：读完每一行之后，你的视觉焦点要从右到左移动一大段距离。

## 1. CSS盒子模型

&nbsp;&nbsp;&nbsp;&nbsp;CSS盒模型本质上是一个盒子，封装周围的HTML元素，它包括：边距，边框，填充，和实际内容。盒模型允许我们在其它元素和周围元素边框之间的空间放置元素，下面的图片说明了盒子模型(Box Model)：

![](/images/CSS/1599105548.jpg)

-   **Margin(外边距)** - 清除边框外的区域，外边距是透明的。
-   **Border(边框)** - 围绕在内边距和内容外的边框。
-   **Padding(内边距)** - 清除内容周围的区域，内边距是透明的。
-   **Content(内容)** - 盒子的内容，显示文本和图像。

>   一般我们不会用Content，而对于其他3种都有上下左右分别对应top、bottom、left、right。
>
>   如果不想单独设置也可以直接用 margin：0px  10px 20px 30px;  这4个分别对应上 右 下 左的顺序。
>
>   也可以 margin：10px 20px;  表示上下外边距各为10px，左右 边距各为20px。。。
>
>   以此类推。

## 2. display显示

&nbsp;&nbsp;&nbsp;&nbsp;display属性设置一个元素应如何显示，visibility属性指定一个元素应可见还是隐藏。

**&nbsp;&nbsp;&nbsp;&nbsp;隐藏一个元素可以通过把display属性设置为"none"，或把visibility属性设置为"hidden"。**

>   两者区别：
>
>   visibility：hidden；隐藏的元素仍需占用与未隐藏之前一样的空间。也就是说，该元素虽然被隐藏了，但仍然会影响布局
>
>   display：none；隐藏的元素不会占用任何空间

display除了可以隐藏元素，还能设置元素的显示方式：

+   **display：inline；  内联显示：元素在一列，不换行**
+   **display：block；  块显示：占用全部的宽度，前后都是换行符**

>   将元素设置为块显示，设置元素的宽度，然后使用外边距 margin：0 auto；可使元素水平居中

~~~css
{
  display: block;
  width: 200px;
  margin: 0 auto;	/*可以根据需要设置上下外边距  margin: 50px auto； */
}
~~~

## 3. position 定位

position 属性的五个值：

+   **static**：默认值，不会受到受到 top, bottom, left, right影响。即没有定位，遵循正常顺序。
+   **fixed**：相对于浏览器窗口是固定位置
+   **relative**：相对定位。相对于其原本的正常位置进行移动
+   **absolute**：绝对定位。相对于最近的已定位的父元素，如果元素没有已定位的父元素，那么它的位置相对于 <html>
+   **sticky**： 基于用户的滚动位置来定位。需要指定 top, right, bottom 或 left 四个阈值其中之一，才可使粘性定位生效。否则其行为与相对定位相同

## 4. z-index 重叠

z-index属性指定了一个元素的堆叠顺序

一个元素可以有正数或负数的堆叠顺序。默认为0，数值大的在上层

~~~css
img{
    position:absolute;
    left:0px;
    top:0px;
    z-index:-1;
}
~~~

## 5. overflow 溢出

overflow：用来设置当元素的内容溢出其区域时发生的事情

overflow-y：指定如何处理顶部/底部边缘的内容溢出元素的内容区域

overflow-x：指定如何处理右边/左边边缘的内容溢出元素的内容区域

常用的属性值如下：

| 属性值     | 描述                           |
| ------- | ---------------------------- |
| visible | 默认值。内容不会被修剪，会呈现在元素框之外。       |
| hidden  | 内容会被修剪，并且其余内容是不可见的。          |
| scroll  | 内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。 |
| auto    | 如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。 |
| inherit | 规定应该从父元素继承 overflow 属性的值。    |

## 6. flex 弹性布局

弹性布局是CSS3的一种新布局模式，是一种当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为的布局方式，也就是自适应响应式。

**弹性容器通过设置 display 属性的值为 flex 或 inline-flex将其定义为弹性容器：**

~~~css
div{
    display: flex;
    justify-content: center;
}
~~~

flex有多个属性，常用的如下介绍。

### 6.1 flex-direction

`flex-direction` 属性指定了弹性子元素在父容器中的位置。用于指定子元素的排序顺序（水平或是垂直）

>   **语法：**
>
>   flex-direction：row | row-reverse | column | column-reverse

~~~css
div{
    display: flex;
    flex-direction: row-reverse；
}
~~~



flex-direction的属性值：

+   row：默认值。水平左起
+   row-reverse： 水平右起
+   column ： 垂直排序，从上往下
+   column-reverse： 垂直排序，从下往上

### 6.2 flex-wrap

`flex-wrap`  决定如果子元素超出父容器时是否换行，该如何换行。

>   语法：
>
>   flex-wrap: wrap | nowrap | wrap-reverse

~~~css
div{
    display: flex;
    flex-wrap: wrap；
}
~~~

flex-wrap的属性值：

+   nowrap：默认值。不换行。
+   wrap：换行，第一行在上方。
+   wrap-reverse：换行，第一行在下方

### 6.3 flex-flow

flex-flow 其实是`flex-direction` 和 `flex-wrap` 的简写。****

>   **例子：**
>
>   flex-flow :  row-reverse  wrap;

### 6.4 justify-content

` justify-content` 用于设置元素在水平方向上的对齐方式。

~~~css
div{
    display: flex;
    justify-content: space-around;
}
~~~

justify-content 的属性值：

+   flex-start：默认值，左对齐
+   flex-end：右对齐
+   center：居中对齐
+   space-between：两端对齐，元素之间留白间隔相等
+   space-around：平均对齐，元素之前、之间、之后留白间隔相等

### 6.5 align-items

`align-items` 用于设置元素在垂直方向上的对齐方式。

~~~css
div{
    display: flex;
    align-items:center;
}
~~~

align-items 的属性值：

+   stretch：默认值，元素被拉伸以适应容器。占满整个容器高度
+   center：中点对齐
+   flex-start：上端对齐
+   flex-end：下端对齐
+   baseline：第一行文字基线对齐

---

&nbsp;&nbsp;&nbsp;&nbsp;这里只是简单介绍了常用的布局所用到的方式和其属性值，具体的用法涉及代码这里没有给出，有需要的可以去具体学习网站。这里推荐 **菜鸟教程 **<https://www.runoob.com/css/css-positioning.html>

