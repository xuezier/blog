---
title: exports 和 module.exports 的区别
date: 2018-07-16 15:56:54
tags:
 - node.js
---
node.js 编写模块的时候，为了保证模块的可读性，推荐把不同功能的代码都写成独立的模块，减少各模块的耦合。

在 node 中， 每个 js 执行文件都会自动创建 module 对象，同时。module 对象默认带有 exports 属性。初始值是 {}。
```js
module.exports = {};
```

在 js 模块里，可以直接调用 module 和 exports 对象。但 exports 是 module.exports 的一个引用。

module.js
```js
var say = function(name) {
  console.log('hello ' + name);
}

exports.say = say;
```

index.js
```js
var module = require('module.js');

module.say('god'); // console print: hello god.
```

在 node 的编译过程中，会把 js 模块进行封装。
```js
// require 是对 Node.js 实现查找模块的 Module._load 实例的引用
// __finename 和 __dirname 是 Node.js 在查找该模块后找到的模块名称和模块绝对路径
(function(exports,require,module,__filename,__dirname){
  function say(name){
    console.log('hello ' + name);
  }
  exports.say = say;
})
```

要将函数直接导出成模块而不是 exports 的一个方法，需要

module.js
```js
var say = function(name) {
  console.log('hello ' + name);
}
module.exports = say;
// wrong example
// exports = say;
```
index.js
```js
var say = require('module.js');
say('god'); // console print: hello god
```
综上，在 js 模块创建的时候
```js
exports = module.exports = {};
```
- exports 是 module.exports 的一个引用
- module.exports 初始值为一个空对象 {}，所以 exports 初始值也是 {}
- require 引用模块后，返回的是 module.exports 而不是 exports
- exports.xxx 相当于在导出对象上挂属性，该属性对调用模块直接可见
- exports = 相当于给 exports 对象重新赋值，调用模块不能访问 exports 对象及其属性
- 如果此模块是一个类，就应该直接赋值 module.exports，这样调用者就是一个类构造器，可以直接 new 实例。

例如
```js
var obj = {
  inner : {}
};
var inner = obj.inner;
```
inner 就是 obj.inner 的一个引用，改变 inner 的属性会 同时对 obj.inner 的属性进行改变
```js
inner.name= 'god';
console.log(obj);
/**
* {
*   inner: {
*     name: 'god'
*   }
* }
*/
```
但是对 inner 重新赋值，并不会对 obj.inner 造成影响
```js
inner = {name: 'devil'};
console.log(obj);
/**
* {
*   inner: {
*     name: 'god'
*   }
* }
*/
```