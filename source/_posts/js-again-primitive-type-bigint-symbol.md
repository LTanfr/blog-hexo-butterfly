---
title: 再学 JS 之数据类型 —— bigint、symbol
date: 2022-05-06 10:20:04
updated:
tags: 
  - 数据类型
categories: 
  - [前端, JavaScript]
keywords:
description:
top_img:
comments:
cover: https://cdn.jsdelivr.net/gh/ltanfr/blog-hexo-butterfly@gh-pages/img/js-again.jpeg
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---
# bigInt 与 symbol

# bigInt

在一个整数字面量后面加 `n` 的方式定义一个 `bigInt`的变量，或者通过调用函数`BigInt()`创建（不需要使用`new`关键字）。该类型一般用于数据过大时的精确运算中，业务场景有限，这里只做基本介绍。

> `bigInt`类型不能和`number`类型的值进行混合运算，但可以转换为同一类型再进行运算；在进行类型转换时，不能像其他类型一样采用`+value`的形式来将`bigInt`类型的值转换为`number`类型，需使用`Number()`方法显式地进行转换；`BigInt`类型的值在转换为`number`类型的值时可能会丢失精度，建议在值可能大于`2^53`的情况下再使用 `bigInt`类型；同时`bigInt`类型的值不能使用`Math`对象的方法。
> 

```js
typeof 4n === 'bigint'
typeof BigInt(3) === 'bigint'
// 与number类型的值可进行比较

// 宽松相等 严格不相等
3n == 3 // true
3n === 3 // false

2 > 1n // true
3.0 <= 3n // true

// BigInt类型的值进行小数运算时会取整
3n/2n === 1n // true

+3n // Uncaught TypeError: Cannot convert a BigInt value to a number

Math.ceil(5n/2n) // Uncaught TypeError: Cannot convert a BigInt value to a number
```

# symbol

`symbol`类型只能通过调用`Symbol()`函数（不需要使用`new`关键字）来创建。该类型在ES6中被引入，它能够保证每个属性的名字都是独一无二的，这样就从根本上防止了属性名的冲突。

```js
typeof Symbol() === 'symbol'
```

`Symbol`函数可以接收一个字符串作为参数，其作用是作为描述便于分辨。需要注意的是，由于`symbol`类型的值是独一无二的，所以就算是传入相同的字符串的`symbol`也不相等。

```js
const s1 = Symbol('foo')
const s2 = Symbol('foo')

console.log(s1) // Symbol(foo)

s1 == s2 // false
s1 === s2 // false
```

## 内置的Symbol

ES6除了引入`symbol`这一基本类型以外，还提供了11个内置的`symbol`值，它们在语言内部使用，用于暴露语言内部的行为。比如`for...of`循环要求被循环的对象实现了`Symbol.iterator`这个接口，开发者可以通过这个属性自定义在遍历时的行为，在这里就不做过多的展开。

```js
// 以下代码为使用 Symbol.iterator 来实现指针结构

class Node {
  value;
  next;
  constructor(value) {
    this.value = value;
    this.next = null;
  }
  [Symbol.iterator]() {
    let iterator = { next: next };
    let current = this;
    function next() {
      if (current) {
        const value = current.value;
        current = current.next;
        return {
          done: false,
          value: value,
        };
      } else {
        return {
          done: true,
        };
      }
    }
    return iterator;
  }
}

let one = new Node(1);
let two = new Node(2);
let three = new Node(3);

one.next = two;
two.next = three;

for (const n of one) {
  console.log(n);
}
// 1 2 3
```

## 常用方法

### Symbol.for(key)

该方法会根据传入的`key`，在运行时的`全局symbol注册表`中去查找是否存在已经登记了相同`key`值的`symbol`，如果存在，则返回该`symbol`；反之则创建一个新的`symbol`，并将其登记到`全局symbol注册表`中去。这也是该方法与`Symbol`方法的区别，直接使用`Symbol`方法创建`symbol`类型的值不会被放到`全局symbol注册表`中。

```js
const s1 = Symbol.for('foo')
const s2 = Symbol.for('foo')

s1 === s2 // true
```

### Symbol.keyFor(symbol)

该方法需要传入一个`symbol`类型的参数，然后会根据传入的参数在`全局symbol注册表`中查找与该参数相关联的`key`值，并返回`key`值；如果没有找到，则返回`undefined`。

```js
let sym = Symbol();
Symbol.keyFor(sym) // undefined

sym = Symbol.for('foo')
Symbol.keyFor(sym) // 'foo'
```

## symbol的实际应用

### 消除魔法字符串

魔法字符串指的是在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。如在团队开发中业务逻辑代码中直接使用字符串字面量就会使得代码难以维护。再如在React源码中，就有大量的symbol应用。

```js
// The Symbol used to tag the ReactElement-like types. If there is no native Symbol
// nor polyfill, then a plain number is used for performance.
const hasSymbol = typeof Symbol === 'function' && Symbol.for;

export const REACT_ELEMENT_TYPE = hasSymbol
  ? Symbol.for('react.element')
  : 0xeac7;
export const REACT_PORTAL_TYPE = hasSymbol
  ? Symbol.for('react.portal')
  : 0xeaca;
export const REACT_FRAGMENT_TYPE = hasSymbol
  ? Symbol.for('react.fragment')
  : 0xeacb;
export const REACT_STRICT_MODE_TYPE = hasSymbol
  ? Symbol.for('react.strict_mode')
  : 0xeacc;
export const REACT_PROFILER_TYPE = hasSymbol
  ? Symbol.for('react.profiler')
  : 0xead2;
export const REACT_PROVIDER_TYPE = hasSymbol
  ? Symbol.for('react.provider')
  : 0xeacd;
export const REACT_CONTEXT_TYPE = hasSymbol
  ? Symbol.for('react.context')
  : 0xeace;
export const REACT_CONCURRENT_MODE_TYPE = hasSymbol
  ? Symbol.for('react.concurrent_mode')
  : 0xeacf;
export const REACT_FORWARD_REF_TYPE = hasSymbol
  ? Symbol.for('react.forward_ref')
  : 0xead0;
export const REACT_SUSPENSE_TYPE = hasSymbol
  ? Symbol.for('react.suspense')
  : 0xead1;
export const REACT_MEMO_TYPE = hasSymbol ? Symbol.for('react.memo') : 0xead3;
export const REACT_LAZY_TYPE = hasSymbol ? Symbol.for('react.lazy') : 0xead4;
```

### 隐藏属性

这样做的好处是可以有效地避免命名冲突问题，一般在开发需要给第三方使用的公共库时会用到这个技巧。而且`symbol`类型的属性不会参与`for...in`、`for...of`等循环，但需要注意，可以通过`Object.getOwnPropertySymbols()`方法获取指定对象的所有 Symbol 属性名。