---
title: javascript(2) 数据类型(2)
date: 2016-12-22 11:48:48
tags:
  - javascript
---
## Object
Object 类型是一种引用数据类型。它指向内存中被标识引用的一个区块。为一个变量赋予一个引用类型数据，变量将指向改引用数据的原始值，而不创建副本。

不同于字符串的值类型。字符串在赋值时，会创建副本，具体情况：
```js
var a = 'hello';
var b = a;
b += ' world';
console.log(a);     // hello
console.log(b);     // hello world
```

值类型数据为 b 创建了副本，b 的值改变创建了一个新的字符串，不改变 a 的原始值

区别于Object
```js
var a = { a: 'hello' };
var b = a;
b.b = 'world';
console.log(a);     // { a: 'hello', b: 'world' }
console.log(b);     // { a: 'hello', b: 'world' }
```

引用类型数据， var b = a 是将 a 的引用传给 b， a, b 指向同一个内存引用区块，改变 b 的数据实际上是改变内存中指向的区块所存储的内容，因此 a 引用到的数据也跟着改变。

## 引用数据比较

```js
var a = { a: 'hello world' };
var b = a;
a == b;       // true
a === b;      // true
```

因为 b 是使用了 a 的引用，a，b 指向内存的同一区块，所以 a 跟 b 是等价的。但是
```js
var a = { a: 'hello world' };
var b = { a: 'hello world' };
a == b;       // false
a === b;      // false
```

创建了一个新的 Object 数据给 b，虽然 a，b 的字面量是一样的，但是 a 与 b 具有不同的引用，对于引用数据的比较，实际是是比较它们的引用。

引用数据变量实际上只保存了引用的内存指针，使用时，是从指针中往内存检索数据。即使字面量一样，但是引用不同，变量就不是等价的。

## property
对象被看作是一组属性的集合，ECMAScript定义的对象中有两种属性：数据属性和访问器属性。

数据属性是键值对，并且每个数据属性拥有下列特性:

* [Value]]
* [[Writable]]
* [[Enumerable]]
* [[Configurable]]

一个 Object 对象是一组键值对的集合，键和值之间的映射关系，键是一个字符串，值可以是任意数据类型。

function 是一个附带可被调用功能的特殊对象

javascript中，很多原型对象数据类型，都是基于 Object 的派生，如：Date, Array, Map, WeakMap 等。

## Array
数组是通过整数标识键，并与长度进行内联的常规对象。键作为数组的游标，与字符串的索引类似。

通过了解数组的构成，可以模拟出数组的数据结构。

*注意，arguments

在 javascript 中，函数的传参被保存在 arguments 对象中，arguments 是一个类数组对象，并非是数组。它和数组一样，具有整数有序游标，和长度关系。
```js
function x(){
  console.log(arguments);     // 执行到这里会输出 Arguments[1, 2]
  arguments.push(3);          // Uncaught TypeError: arguments.push is not a function
}
x(1, 2);
```

展开 Arguments[1, 2]，可以看到它的数据结构是一个整数游标键，并具有 length。与数组结构相似。但并不是一个数组，所以不具备数组的方法。

javascript 可以通过 call 和 apply 方法继承。
```js
function x(){
  Array.prototype.push.call(arguments, 3)
  console.log(arguments);     // 执行到这里会输出 Arguments[1, 2, 3]
}
x(1, 2);
```
构建一个类数组对象

```js
var a = { 0: 'a', 1: 'b', 2: 'c', length: 3 };
Array.prototype.forEach.call(a, function(value, index){
  console.log(value,',',index);     // 会依次输出 a, 0   b, 1   c, 2
});
```
