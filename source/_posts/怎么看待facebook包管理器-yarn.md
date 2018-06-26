---
title: 怎么看待facebook包管理器 yarn
date: 2016-10-26 11:29:47
tags:
  - yarn
---
![](http://ofn8y0v16.bkt.clouddn.com/Yarn-796x398.jpg)

Yarn 是一个Facebook推出的新的包管理器，用于替代现有的 npm 客户端或者其他兼容 npm 仓库的包管理工具。Yarn 保留了现有工作流的特性，优点是更快、更安全、更可靠。

对于npm，最大的缺点就是慢，特别是对于国内的开发者来说，墙外的世界总是爱不释手又遥不可及。一个大点的依赖包可能需要近半个小时才能down下来。还好我们还有cnpm，还有taobao的镜像源。

yarn到底有多快，能不能取代npm安装，下面通过安装几个模块来验证一下。

## 安装相同的包express所耗费的时间
为了不浪费时间做无谓的测试，就不直接测试npm的源了，而是将npm的源更换为我已经搭好的国内镜像服务。

  yarn使用默认源，npm使用国内镜像
```js
// 因为npm不能直接输出安装时间，所以我们选择通过代码检测整个程序的穿透时间来做测量
'use strict';
let child_process = require('child_process');
function test(m){
  console.time(`${m} finished in :`);
  child_process.exec(`${m == 'npm' && (m + ' install') || (m + ' add')} express`,function(){
    console.timeEnd(`${m} finished in :`);
  });
}
test('npm');  // finished in 4331.464ms
test('npm');  // finished in 1809.988ms
// 清空掉安装历史，继续执行以下代码
test('yarn');  // finished in 2810.565ms
test('yarn');  // finished in 1106.524ms
```
  将yarn安装源换成与npm一样的国内镜像
```bash
// 为了测试整个项目的可用性，挑选了一个依赖较多的项目来做干净安装测试
yarn install  // 43.27s
npm install   // >100s
// 该测试都是在使用同一源的情况下进行
```

  安装在我私有源上的私有包
```js
// 在开发的时候，我们搭建私有npm代理服务的目的也是为了管理只有自己使用的一些内部扩展包，如果不支持私有包的安装，yarn就无法取代传统的npm
'use strict';
let child_process = require('child_process');
function test(m){
  console.time(`${m} finished in :`);
  child_process.exec(`${m == 'npm' && (m + ' install') || (m + ' add')} @ym/rpc`,function(){
    console.timeEnd(`${m} finished in :`);
  });
}
test('npm');  // finished in 7547.653ms
test('npm');  // finished in 5121.880ms
// 清空掉安装历史，继续执行以下代码
test('yarn');  // finished in 2752.662ms
test('yarn');  // finished in 1140.099ms
```

yarn在即使不使用国内镜像访问的时候，安装公有包的速度依然比使用国内镜像的npm快，在使用同是国内镜像源的时候，优势更是明显。

### 测试安装整个项目依赖
```bash
// 为了测试整个项目的可用性，挑选了一个依赖较多的项目来做干净安装测试
yarn install  // 43.27s
npm install   // >100s
// 该测试都是在使用同一源的情况下进行
```

## focus
以上安装测试每个都至少测试3次，每次测试都需清空掉安装信息，平均安装时间误差波动不大

## 综述
yarn用作包管理器，在速度上确实有很大优势，也支持私有npm包管理安装。

除了有时候会有不稳定的情况出现，整体上还是很不错的，只要稍加优化，作为npm的替代品是没有问题
