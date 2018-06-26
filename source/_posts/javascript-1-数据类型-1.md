---
title: javascript(1) 数据类型(1)
date: 2016-12-22 11:42:53
tags:
  - javascript
---

ECMAScript 定义了7种标准的数据类型

* 6种基本数据类型
  * Boolean
  * Null
  * Undefined
  * Number
  * String
  * Symbol (es6 标准新增)
* 1种特殊数据类型
  * Object

## 原始值
除了 Object 以外的数据类型，值本身是不可变的。例如：对字符串进行操作，返回了一个新字符串，而原始字符串本身并没有被改变。

所以除 Object 类型变量外，要改变变量的值，需要重新赋值。
```js
var str = 'hello';
str + ' world';   // 表达式返回 'hello world'
console.log(str); // 输出 'hello', str 本身的值没有被改变
str += ' world';  // 表达式返回 'hello world'
console.log(str); // 输出 'hello world', str 被重新赋值
```

## Boolean
Boolean 表示一个逻辑实体，有两个值：true, false。

所有的值都可以解释成布尔值，除了一些特定值被解释成 false，其他值都将解释成 true。
* false : false, undefined, null, 0, ‘’(空字符串), NaN
注意：这些值虽然可以解释成 false，但并不表示就与 false 等价：
```js
undefined == false;   // false
undefined === false;  // false
null == false;        // false
null === false;       // false
0 == false;           // true
0 === false;          // false
'' == false;          // true
'' === false;         // false
NaN == false;         // false
NaN === false;        // false
```

## Null
null 是 javascript 的一个字面量，表示空。 null 常用做表示期望对象，但是没有任何引用。

## Undefined
通常所说的 undefined 是指全局对象的一个属性 window.undefined。

未初始化的变量都有一个默认值 undefined，未被传入实参的形参值为 undefined，未添加 return 返回值的函数，默认返回 undefined。
```js
var x;                  // undefined
function x1(a){
  console.log(a);
  // without return
}
x1();                   // return undefined, and console print a as undefined.
```

null 与 undefined 比较
```js
typeof null;          // object, 空引用对象，有些人觉得这应该算是一个 bug
typeof undefined;     // undefined
null == undefined;    // true
null === undefined;   // false
```

判断一个值是否是 undefined：
```js
var x;
x === undefined && 1 || 0;            // 1
typeof x === 'undefined' && 1 || 0;   // 1
typeof y === 'undefined' && 1 || 0;   // 1
y === undefined &&1 || 0;             // ReferenceError, y 未被声明过
```

## Number
数值类型的取值范围 -(2^53 -1) 到 2^53 -1 (SAFE_INTEGER_VALUE)。javascript 数值类型没有类似 java 分为整型，长整型，浮点型，双精度浮点型。

javascript 中所有数值都是 Number 类型。除了数值外，Number还包括一些带符号的值：+/-Infinity, NaN(Not-a-Number)。

*注意：0 === -0 为真， 但在计算时：
```js
1/0;      // Infinity
1/-0;     // -Infinity
```

## String
字符串类型用于表示文本数据，它是一个带索引的元素集合，字符串中每个元素都占据一个位置，第一个元素的索引是 0，往后递增。

字符串一旦被创建，就不会被更改，字符串操作会返回一个新的字符串，但不会改变原始字符串。

表达文本数据和符号数据时候推荐使用字符串。当表达复杂的数据时，使用字符串解析和适当的缩写。

*注意：对于数值字符串和数值型数据：
```js
'1' == 1;   // true
'1' === 1;  // false
```
所以在对一些数据判断时，无法保证传入的是数值型字符串或数值型数据，根据使用场景判断是否使用严格相等操作符。
