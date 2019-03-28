---
title: CSS3 border-radius
date: 2017-01-18 13:26:11
tags:
  - css
  - css3
---
早些年CSS3 border-radius 的文档或者文章，会发现，根本就没有提过百分比值这一茬。究其原因，是因为百分比值的支持是后来才支持的，
跟数值不是一起出来的。比方说某些老版本的 Android 机子，border-radius:50% 它就不认识。

早期浏览器支持 border-radius 都是浏览器标准实现，并不形成统一标准。兼容旧版本浏览器需要用到浏览器内核前缀 -webkit-border-radius, -moz-border-radius。
浏览器个把年前就把私有前缀给去掉了，现在完全没必要使用。

## 作用
border-radius 是向块级元素添加圆角边框。

## 原理
既然是添加圆角边框，那么圆角边框是怎么被确定出来的，为什么这么多。

首先确定圆角边框其实就是一个弧边。一个标准的圆角弧边是由圆裁切出来的。圆的半径决定了圆的大小，也就决定了弧边裁剪出来的弧长和弧度。

## border-radius 绝对数值
border-radius 的值是绝对像素数值 apx(px是单位像素) 的时候，切出来的弧边正好是一个以 apx 为半径的圆的 1/4 圆角弧边。

例如，一个宽高为 200px 的正方形 div， 设置 50px 的 border-radius，圆角边框就是 50px 为半径的圆的 1/4 圆角弧边
```html
<style>
.square{
	width: 200px;
	height:200px;
	border: 1px solid #000;
	margin-right: 100px;
	display:inline-block
}
.radius{
	border-radius: 50px;
}
</style>
<div class='square'></div>
<div class='square radius'></div>
```
![](http://cdn-public.imxuezi.com/radius01.png)

当 apx 为正方形 div 边长的一半时，div 正好是一个正圆
```css
.radius{
	border-radius: 100px;
}
```
![](http://cdn-public.imxuezi.com/radius02.png)

apx 的最大值是元素边长的一半, 当 apx 超过一半时，将自动取边长的一半
```css
.radius{
	border-radius: 150px;
}
```
![](http://cdn-public.imxuezi.com/radius02.png)

如果 div 是一个长方形，那么 border-radius 的取值最大是最短边的一半
```html
<style>
.square{
	width: 400px;
	height:200px;
	border: 1px solid #000;
	margin-right: 100px;
	display:inline-block
}
.radius{
	border-radius: 150px;
}
</style>
<div class='square'></div>
<div class='square radius'></div>
```
![](http://cdn-public.imxuezi.com/radius03.png)

```css
.square{
	width: 200px;
	height:400px;
	border: 1px solid #000;
	margin-right: 100px;
	display:inline-block
}
```
![](http://cdn-public.imxuezi.com/radius04.png)

## border-radius 百分比值
现代浏览器为 border-radius 加入了半分比值的支持。半分比值生成的圆角边框，是从元素顶点处，往两个邻边取出边长的一半的半分比长度所确定的椭圆的 1/4 弧边
首先我们了解，标准椭圆的确定概念

椭圆（Ellipse）是平面内到定点F1、F2的距离之和等于常数（大于|F1F2|）的动点P的轨迹，F1、F2称为椭圆的两个焦点。其数学表达式为：|PF1|+|PF2|=2a（2a>|F1F2|）。
![](http://cdn-public.imxuezi.com/border-radius.jpg)

```html
<style>
.square{
	width: 200px;
	height:200px;
	border: 1px solid #000;
	margin-right: 100px;
	display:inline-block
}
.radius{
	border-radius: 20%;
}
</style>
<div class='square'></div>
<div class='square radius'></div>
```
![](http://cdn-public.imxuezi.com/radius05.png)

F1, F2 的位置是浏览器给出，因此很难说其切面弧角的大小。

百分比值最大取到 50%，即为各边一半，超过 50% 按照 50%计算。
```css
.radius{
	border-radius: 50%;
}
```
![](http://cdn-public.imxuezi.com/radius06.png)

```css
.radius{
	border-radius: 80%;
}
```
![](http://cdn-public.imxuezi.com/radius06.png)

## 多个参数
border-radius 支持传入多个参数进行设置圆角边框，只有一个参数时表示4个角的圆角是一样的，多个参数时即为每个角单独设置，顺序是：左上，右上，右下，左下
```html
<style>
.square{
	width: 200px;
	height:200px;
	border: 1px solid #000;
	margin-right: 100px;
	display:inline-block
}
.radius{
	border-radius: 10px 20px 20px 20%;
}
</style>
<div class='square'></div>
<div class='square radius'></div>
```
![](http://cdn-public.imxuezi.com/radius07.png);
