---
title: typescript - decorator(1)
date: 2019-03-01 16:36:41
tags:
 - typescript
 - javascript
 - decorator
---
## 介绍

    With the introduction of Classes in TypeScript and ES6, there now exist certain scenarios that require additional features to support annotating or modifying classes and class members. Decorators provide a way to add both annotations and a meta-programming syntax for class declarations and members.

翻译：

    随着 typescript 和 es6 中类的引入，现在存在一些场景，需要额外的功能来支持对类和类成员进行修饰或修改。decorator(装饰器) 提供了一种为类声明和成员添加修饰和元编程语法的方法。

## 装饰器

装饰器是一种特殊类型的声明，它能够被附加到类声明，方法， 访问符，属性或参数上。 装饰器使用 @expression 这种形式，expression 求值后必须为一个函数，它会在运行时被调用，被装饰的声明信息做为参数传入。

装饰器的作用是为类声明和成员添加修饰和元编程语法。首先要了解，什么样的场景下需要做这样的装饰器。

有一个类 Person:
```typescript
class Person {
    say(sentence) {
        console.log(sentence);
    }
}
```
Person 有一个实例方法 say，可以说话。但有一个问题，

假如 Person 的实例 mick 喜欢说脏话，比如 cnm 这样的。那么我想让 mick 沉默，或者 cnm 就变成 bi~~~~。

通常的做法是给 say 方法加一些内容:
```typescript
class Person {
    say(sentence) {
        sentence = sentence.replace(/cnm/g, 'bi~~~');
        console.log(sentence);
    }
}
const mick = new Person();
mick.say();
```
用装饰器实现
```typescript
// workFilter 是一个 decorator
function wordFilter(target: any, name: string, descriptor: PropertyDescriptor) {
    const fun = descriptor.value;
    descriptor.value = function(sentence) {
        sentence = sentence.replace(/cnm/g, 'bi~~~');
        return fun.call(this, sentence);
    }
}

class Person {
    @wordFilter
    say(sentence) {
        console.log(sentence);
    }
}

const mick = new Person();
mick.say();
```

简单理解装饰器就是在原有的代码上包装了一层，

可以想像手枪的枪口装了消音器，水龙头的进水管加装了净水器一样，不改变原有的实现逻辑，增加原来不存在的功能或特性。

## 装饰器种类
装饰器主要分为类装饰器和方法装饰器，分别用来修饰类声明和成员。

### 类装饰器
类装饰器只有一个参数
- target 类的构造函数


### 方法装饰器
方法装饰器有三个参数
- target any, 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
- name string, 成员的名字
- descriptor TypedPropertyDescriptor<T>, 成员的属性描述符
    - value: any 成员的值
    - writable: boolean 是否修改
    - enumerable: boolean 是否可遍历
    - configurable: boolean 成员描述是否可改变或者成员是否可删除
    - get 成员的 getter 方法
    - set 成员的 setter 方法

### 装饰器工厂
装饰器工厂就是一个简单的函数，它返回一个表达式，以供装饰器在运行时调用。
```typescript
function wordFilter(word) {
    const reg = new RegExp(word, 'g');

    return function wordFilter(target: any, name: string, descriptor: PropertyDescriptor) {
        const fun = descriptor.value;
        descriptor.value = function(sentence) {
            sentence = sentence.replace(reg, 'bi~~~');
            return fun.call(this, sentence);
        }
    }
}

@wordFilter('cnm')
class Person {
    @wordFilter
    say(sentence) {
        console.log(sentence);
    }
}
```

## 参考
- [【Typescript - Decorators】](https://www.typescriptlang.org/docs/handbook/decorators.html)
