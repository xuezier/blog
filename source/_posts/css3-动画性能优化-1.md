---
title: css3 动画性能优化(1)
date: 2017-01-23 13:33:03
tags:
  - css
  - css3
  - idea
---

在传统网页上动画一般都是通过 Javascript 或 flash 来实现，但是 html5 的时代的到来，CSS 的进化，让动画实现起来更加 easy。

现代的浏览器通常会有两个重要的执行线程，这2个线程协同工作来渲染一个网页
* 主线程
* 合成线程

一般情况下，主线程负责：
* 运行 JavaScript。
* 计算 HTML 元素的 CSS 样式。
* 页面的布局
* 将元素绘制到一个或多个位图中
* 将这些位图交给合成线程

相应地，合成线程负责：
* 通过 GPU 将位图绘制到屏幕上
* 通知主线程更新页面中可见或即将变成可见的部分的位图
* 计算出页面中哪部分是可见的
* 计算出当你在滚动页面时哪部分是即将变成可见的
* 当你滚动页面时将相应位置的元素移动到可视区域

## Example width 过渡
我们绘制一个新闻卡片，让其在鼠标移上时产生动画，为了节省时间，css 采用 sass/scss 编译

html 代码：
```html
<div class="box">
  <div class="img-box">
    <img src="./images/baobao.jpeg" />
  </div>
  <div class="content">
    <p>hello world</p>
    <p>hello world</p>
    <p>hello world</p>
    <p>hello world</p>
    <p>hello world</p>
  </div>
</div>
```
scss 代码：
```scss
.box {
  width: 120px;
  box-shadow: 0 0 10px 0 #777;
  @include comic(.3);
  &:hover {
    -webkit-animation: box 1s forwards;
    .content {
      -webkit-animation: content 1s forwards;
      -webkit-animation-delay: .5s;
      p {
        -webkit-animation: p 1s forwards;
        -webkit-animation-delay: .5s;
      }
    }
  }
  border-radius: 50%;
  overflow: hidden;
  .img-box {
    width: 100%;
    img {
      width: 100%;
    }
  }
  .content {
    padding: 0;
    p {
      height: 0;
      margin: 0;
      line-height: 20px;
    }
  }
}
@keyframes box {
  0% {}
  100% {
    width: 200px;
    border-radius: 0;
  }
}
@keyframes content {
  0% {}
  100% {
    padding: 10px;
  }
}
@keyframes p {
  0% {}
  100% {
    height: 20px;
    margin: 1em 0;
  }
}
@mixin comic($time) {
  -webkit-transition: #{$time}s ease-in-out;
  -moz-transition: #{$time}s ease-in-out;
  -ms-transition: #{$time}s ease-in-out;
  -o-transition: #{$time}s ease-in-out;
  transition: #{$time}s ease-in-out;
}
```
鼠标悬停时，会产生如下效果
![](http://ofn8y0v16.bkt.clouddn.com/css-animation01.gif)

打开 chrome 浏览器的 timeline 观看整个动画绘制情况
![](http://ofn8y0v16.bkt.clouddn.com/animate04.png)

从 range 可以看出 redering 和 painting 所花费的时间情况，整个动画稳定在 55-60 fps 左右。这是在配置较好的笔记本上的情况，尝试在
较为旧款的设备上查看，有时会降为 40 fps 左右，略有卡顿。

容器宽度（width）动画流程大致如下：
![](http://ofn8y0v16.bkt.clouddn.com/design01.png)

左线程为主线程，右线程为合成线程

如图所示，浏览器在进行一次 css 动画的时候，需要将动画进行切片，每进行一次帧过渡，都需要主线程与合成线程进行协调。
在动画的每一帧中，浏览器都要执行布局、 绘制、 以及将新的位图提交给 GPU。我们知道，将位图加载到 GPU 的内存中是一个相对较慢的操作。

浏览器需要做大量工作的原因在于每一帧中元素的内容都在不断改变。改变一个元素的高度可能导致需要同步改变它的子元素的大小，
所以浏览器必须重新计算布局。布局完成后，主线程又必须重新生成该元素的位图。浏览器需要做大量的工作，也就是说这个动画可能会变得卡顿。

## scale 过渡
将 width 的数值过渡改成缩放过渡
```scss
.box {
  width: 200px;
  box-shadow: 0 0 10px 0 #777;
  @include comic(.3);
  -webkit-transform: scale(.6);
  &:hover {
    -webkit-animation: box 1s forwards;
    .content {
      -webkit-animation: content 1s forwards;
      -webkit-animation-delay: .5s;
      p {
        -webkit-animation: p 1s forwards;
        -webkit-animation-delay: .5s;
      }
    }
  }
  border-radius: 50%;
  overflow: hidden;
  .img-box {
    width: 100%;
    img {
      width: 100%;
    }
  }
  .content {
    padding: 0;
    p {
      height: 0;
      margin: 0;
      line-height: 20px;
    }
  }
}
@keyframes box {
  0% {}
  100% {
    -webkit-transform: scale(1);
    border-radius: 0;
  }
}
@keyframes content {
  0% {}
  100% {
    padding: 10px;
  }
}
@keyframes p {
  0% {}
  100% {
    height: 20px;
    margin: 1em 0;
  }
}
```
同样打开 chrome 浏览器的 timeline 观看整个动画绘制情况
![](http://ofn8y0v16.bkt.clouddn.com/animate01.png)

会发现 rendering 和 painting 的时间都缩短了。
![](http://ofn8y0v16.bkt.clouddn.com/animate03.png)

同时在时间轴上可以看到最大可以达到 90 几 fps（有时可以达到100以上）

容器宽度（width）动画流程大致如下：
![](http://ofn8y0v16.bkt.clouddn.com/design02.png)

我们可以看到少了很多主线成与合成线程之间的交换，意味着动画变得更流畅了。

## 原因
根据定义，CSS 的transform属性不会更改元素或它周围的元素的布局。transform属性会对元素的整体产生影响，
它会对整个元素进行缩放、旋转、移动处理。

而改变 width, height, maigin 等属性的时候，会造成元素布局的重排而触发重绘，影响 gpu 的线性计算。

这只是一个相对简单的动画实例，对比情况并不明显，在涉及内连元素较多的动画同时进行时，对比会更为突出。

## 最后
进行用以下属性来做 css 动画
* CSS transform
* CSS opacity
* CSS filter（取决于filter的复杂度以及浏览器的实现）
避免使用 height,width,margin,padding 等；
