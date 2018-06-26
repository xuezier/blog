---
title: '利用scss方法构建像素化形象 '
date: 2017-06-10 13:49:14
tags:
  - css
  - css3
  - scss
---
像素是一个一个色块组成的，像素化顾名思义就是将图像分成一定的区域，并将这些区域转换成相应的色块，再有色块构成图形。类似于色彩构图。简单说就是将图像转换成像素点组成的点阵图形，也叫栅格化。

## 准备
绘制像素化图像，我们先要将形象进行像素化，将像素化后的点阵图像描述出来，就是颜色矩阵
![](http://ofn8y0v16.bkt.clouddn.com/pikatu.png)

为此，我们先分析图像上的颜色，共有四种颜色 #fefe00(黄), #823e00(褐), #c69(红), #000(黑)

我们将图像的颜色用矩阵描述出来
```scss
// scss代码
// 颜色命名
$colors: (
  y: #fefe00,
  h: #823e00,
  r: #c69,
  k: #000
);
// 图像矩阵
$pixel-art:(
  $pikachu:(
    (o o o o o o o o o o o o o o o)
    (o o o o k k o o o o o o o o o)
    (k k k k k k k k k o o o o o o)
    (k k k k k k k y y k k k o o o)
    (o k y y y k y y y y y y k o o)
    (o o k y y k k y y y y y y k o)
    (o o o k y y k y y y y y y k o)
    (o k k k y k y y y y y k y y k)
    (o k y y y k y y y r y y y y k)
    (o o k h k h h h y y y y y k o)
    (o k h h k y y y y y y k y k o)
    (o o k h k h h h y y k k y k o)
    (o o o k k k y y y y y y y k o)
    (o o o o o o k k y y y k k o o)
    (o o o o o o o o k k k o o o o)
  )
)
```

以上就是我们的皮卡丘的颜色矩阵了，现在只要将矩阵上的每一个点像素化就可以了

## 像素化
box-shadow 可以为一个元素添加多个阴影块，我们把每个像素当作一块阴影进行填充，用scss中的函数遍历矩阵，将每个颜色点像素化
```scss
// scss 代码
 // 定义每个像素的大小，宽高等值
$pixel-size: 4px !default;
// 像素化函数
@function pixelize($matrix, $size) {
  $l: length($matrix);  // 矩阵长度
  $sh: '';              // 阴影值
  @for $row from 1 through $l { // 遍历每一行
    $row-num: nth($matrix,$row); // 获取每一行有多少点
    @for $col from 1 through length($row-num) { // 遍历每一行的节点
      $dot: nth($row-num, $col);    // 点的位置
      $sh: $sh+ ($col*$size)+' '+($row*$size)+' '+map-get($colors,$dot);  // 填充该点的阴影值
      @if not ($col == length($row-num) and $row==$l){
        $sh:$sh+',';
      }
    }
  }
  @return unquote($sh);
}
@mixin style-item($matrix,$size){
  width: $size*length(nth($matrix,1)); // 容器的宽，与像素化图形等宽
  height: $size*length($matrix);       // 容器的高，与像素化图形等高
  &:after {
    content: '';
    position: absolute;
    top:-$size;
    left: -$size;
    width: $size;
    height: $size;
    box-shadow: pixelize($matrix, $size); // 填充阴影
  }
}
// 遍历我们的每一个像素化图形，将每个图形的阴影放到对应的 class 上
@each $key,$value in $pixel-art {
  .#{$key}{
    @include style-item($value,$pixel-size);
  }
}
```

## 结束
接下来我们要做的就是，编译这些 scss 代码到 css 文件，可以用 gulp,babel 等工具, 最后，我们在 html 中引入这个 css 文件，并绑定对应的图形 class
```html
<html>
  <head>
    <link href='path/to/your.css' rel='stylesheet' />
  </head>
  <body>
    <div class='pikachu'></div>
  </body>
</html>
```

一只像素化的皮卡丘就跑到我们页面上了
