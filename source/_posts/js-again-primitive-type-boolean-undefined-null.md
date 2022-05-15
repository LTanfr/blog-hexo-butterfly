---
title: 再学 JS 之数据类型 —— boolean、undefined、null
date: 2022-05-05 19:35:21
updated:
tags: 
  - 数据类型
categories: 
  - 前端
  - JavaScript
keywords:
description:
top_img:
comments:
cover: https://cdn.jsdelivr.net/gh/ltanfr/ltanfr.github.io//img/js-again.jpeg
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
# boolean

原始类型中的`boolean`类型仅有两个值：`true`和`false`，即真与假。通常使用布尔类型的值来进行一些逻辑判断，需要注意的是其中常常会存在类型转换和逻辑运算符的使用。

```js
typeof true === 'boolean' // true

// 逻辑运算符的使用与隐式类型转换
typeof !'false' === 'boolean' // true
```

## 有关boolean类型转换

### 显式类型转换

可以使用之前提到过的对象包装器`Boolean`将其他类型的值进行显示地转换为boolean的值，或者使用双重非（`!!`）。

```js
const bool1 = !!{x:100} // 将对象显式转换为boolean值
const bool2 = Boolean('Yui') // 将字符串显式转换为boolean值
// 使用new关键字时创建的是一个对象，而不是类型转换
const boolObj = new Boolean(false)

typeof bool1 // 'boolean'
typeof bool2 // 'boolean'
typeof boolObj // 'object'
```

### 隐式类型转换

类型转换指的是将其他类型的值转换为`boolean`的值时所采用的规则。如在使用`Boolean`转换其他类型的值时，未传入参数或者参数值为`0`、`null`、`undefined`、`NaN`、`''`时转换后的布尔值为`false`。

因为对象的类型转换规则比较复杂，详细的规则会在之后讲到。

```js
// 一个涉及对象的隐式类型转换的例子
[] == ![] // true
```

隐式类型转换指的是在某些情况下会自动地将非布尔类型的值转换为布尔类型的值。
# undefined

原始类型中的`undefined`只包含一个值：`undefined`，意为未被赋值。当使用`var`、`let`关键字声明变量而赋值时，变量的值就为`undefined`，同时也可以显示地给变量赋值为`undefined`（不建议这样做）。

```js
typeof undefined // 'undefined'

let undefinedX
typeof undefinedX // 'undefined'
```

# null

null同样也只包含一个值：null，意为空值。需要注意的是使用typeof判断null的类型时会与期待的结果有所不同。

```js
typeof null // 'object'
```

虽然`null`是原始类型的值，但是在这里却被“错误”地判断为`object`，这实际上是远古时期JavaScript就存在的一个bug，却一直没有得到修复，具体原因是JavaScript中，不同的对象在底层的表示都为二进制，二进制表示前三位为都为`0`的话就会被认为是`object`，而`null`的二进制表示全为`0`，故被错误地识别为`object`。

## undefined 与 null 的区别

从逻辑上讲，`null`处于主动，需要使用时必须显式地赋值；`undefined`处于被动，当使用者忘记赋值时系统会默认赋值为`undefined`，如函数没有返回值时会默认返回`undefined`、调用函数未传入应该提供的参数时参数默认值为`undefined`等。

另外需要注意的是`undefined`在进行类型转换时与`null`的区别。

```js
// 涉及隐式类型转换
// 关系运算符与相等运算符的规则不同
// 在设计上关系运算符总会将运算元尝试转换为number，而相等运算符则没有这方面的考虑
undefined == null // true
10 + null // 10
10 + undefined // NaN

null > 0  // null会被转换为0， 结果为false
null == 0 // null在此处不会进行类型转换 结果为false

Number(null) // 0
Number(undefined) // NaN

// 使用全等运算符判断结果为false
// 关于全等运算符与相等运算符的差别后面会讲
undefined === null // false
```

# 参考资料

**JavaScirpt高级程序设计（第4版）**

**你不知道的JavaScript**

**ECMAScript 6 入门**

**[现代 JavaScript 教程](https://zh.javascript.info/)**

**[MDN Web Docs](https://developer.mozilla.org/zh-CN/)**