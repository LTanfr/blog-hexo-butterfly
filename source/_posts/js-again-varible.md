---
title: 再学 JS 之变量
date: 2022-05-05 14:02:00
updated:
tags: 
  - 变量
categories: 
  - [前端, JavaScript]
keywords:
description:
top_img:
comments:
cover: https://fastly.jsdelivr.net/gh/ltanfr/blog-hexo-butterfly@gh-pages/img/js-again.jpeg
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
# 作用域简述

提到变量，就离不开作用域，这里先简单提及一下，具体的会放到后面讲。作用域指的是当前执行的上下文，也可以理解为作用范围。子作用域可以访问父作用域，而父作用域不能访问子作用域（涉及作用域链）。

JavaScript中作用域分为全局作用域、函数作用域和块级作用域（ES6新增）。顾名思义，全局作用域的作用范围为全局，函数作用域为当前的函数，块级作用域为所在的代码块`{...}`。在这里先粗略地知道这三个概念就可以了。

# JS变量


在JavaScript中，变量为松散类型，可以用于保存任何类型的数据。

一共有三个关键字用于声明变量：`var`、`let`、`const`。其中`let`、`const`为ES6时新增的关键字，只能在ES6之后的版本使用。

从另一个角度来说，使用`function`、`import`、`class`关键字也可以声明变量，但这里不会提及。

需要注意的是，在非严格模式下，不使用关键字声明的变量为全局变量，会被挂到全局对象上，在浏览器环境下为`window`。不过这样做的话会不仅会导致变量难以维护，而且变量无法被回收，污染了全局对象，造成了内存泄漏。

```js
if(true) {
	a = 10
}

// 浏览器环境下
this === window // true
console.log(this.a) // 10
```

需要定义多个变量时可以使用`,`号分隔每个变量。

```js
var str1 = 'c++', str2 = 'java', str3 = 'c#'
```

# 变量命名

JavaScript 的变量命名有两个限制：

1. 变量名仅包含字母，数字，符号 `$`
 和 `_`。
2. 首字符必须非数字。

> 除此之外，JavaScript中的关键字也不能用来命名变量，但并不是所有的关键字，比如`async`仍可以用来命名，不能用来命名的关键字可以参考[保留关键字](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Lexical_grammar#ecmascript_6_%E4%B8%AD%E7%9A%84%E4%BF%9D%E7%95%99%E5%85%B3%E9%94%AE%E5%AD%97)，为了以防在未来的版本中关键字被加入到保留关键字中，最好不要使用关键字对变量命名。
> 

```js
const 1m = 2 // Uncaught SyntaxError: Invalid or unexpected token
var t-an; // Uncaught SyntaxError: Unexpected token '-'

let 这是一个变量名 = 22 // 合法，但不推荐
```

现在主流的命名方式是[小驼峰](https://zh.wikipedia.org/wiki/%E9%A7%9D%E5%B3%B0%E5%BC%8F%E5%A4%A7%E5%B0%8F%E5%AF%AB)式命名法，即首个单词的首字母小写，从第二个单词开始首字母大写。

常量用作别名时，命名时通常会将单词大写，单词与单词之间使用`_`链接。

```js
let currentYear = 2022
const COMPANY_NAME = 'it is too looooooooooooooong to remember'
```

# var 关键字

在声明未赋值的情况下变量会保存一个特殊值`undefined`。

```js
var msg // 声明
console.log(msg) // undefined
```

可以重新赋值，不仅可以改变值，也可以改变值的类型，但并不推荐这样做，容易引起逻辑混乱。

```js
var msg = 100 // 声明变量并进行赋值
msg = 'Hi' 
```

重复声明不会报错，但是会被认为是无效声明，但是赋值是有效的。

```js
var test = true // 声明并赋值
var test = false // 无效的重复声明
console.log(test) // false
```

通过var声明的全局变量也会挂到全局对象上去。

```js
var aaa = 'aaa'

// 浏览器环境下
window.aaa === 'aaa' // true
```

## 作用范围

由于在ES6之前并没有块级作用域与全局作用域之分，且声明变量的关键字也只有`var`，因此通过关键字`var`声明的变量的有效作用范围为函数作用域。

在函数作用域中使用`var`关键字定义的变量只能在其定义的函数中访问，在作用范围外调用会报引用错误。

```js
function fn1 () {
	var n = 100 // 函数作用域中定义的局部变量
}
fn1();
console.log(n) // Uncaught ReferenceError: msg is not defined
```

## 变量提升

变量提升指的是会将变量的声明提升到当前作用域的顶部。用`var`关键字定义的变量就存在变量提升，注意**提升的只是变量的声明，并不是赋值**。

```js
console.log(age) // undefined
var age = 99

function test() {
	// 变量提升，变量a先被赋值为1，再被赋值为0
	a = 1;
	var a = 0;
	console.log(a) // 0
}
```

## 模仿块级作用域

由于在ES6之前并不存在块级作用域，但是上有政策下有对策，程序员们发明了“立即调用函数表达式”（immediately-invoked function expressions，IIFE），即创建一个函数并立刻调用它来实现块级作用域的效果，一般的写法为`(function(){...})()`，尽管在ES6块级作用域出现后，这种方法就不再使用了，但有时候还是会遇到这种写法。

```js
(function(){
	var temp = 20;
	console.log(temp)
})()

// 以下的写法也可以
+function(){
	var temp = 20;
	console.log(temp)
}()

(function(){
	var temp = 20;
	console.log(temp)
}())

```

# let 关键字

## 作用范围

这里主要讲一下`let`与`var`的区别，let作为块级作用域变量，只能在其声明的作用域或子域中使用，而在块级作用域中使用`var`关键字声明的变量在块级作用域之外仍能够访问。

```js
{
	let a = 1
	var b = 2
}

console.log(a) // Uncaught ReferenceError: a is not defined
console.log(b) // 2

for(var i = 0; i < 5; i++) {
  // 不受块级作用域约束，迭代变量渗透到循环体外部
	// 相当于重复声明i, 并对其赋值
	// 输出5个5
	setTimeout(()=>{
		console.log(i)
	})
}

for(let j = 0; j < 5; j++) {
	// 受到块级作用域的约束
	// 每一次循环开始时都在新的块级作用域声明，保留其块级作用域中引用的值
	// 输出 0 1 2 3 4
	setTimeout(()=>{
		console.log(j)
	})
}
```

同时使用`let`关键字声明的变量不能在同一个块级作用域中重复声明，这样会导致报语法错误。

```js
var time;
let time; // Uncaught SyntaxError: Identifier 'time' has already been declared

let age;
var age; // Uncaught SyntaxError: Identifier 'age' has already been declared

let name = 'Joe'
if(true) {
	let name = 'Roselyn'
	console.log(name) // Roselyn
}
console.log(name) // Joe

```

当使用`let`关键字在全局作用域声明变量时也不会污染全局对象。

```js
let a = 10;
// 浏览器环境下
this.a // undefined
```

## 暂时性死区（temporal dead zone，简称 TDZ）

使用`let`声明的变量不会被提升到当前作用域的顶部，直到它们的定义被执行时才可以访问。在`let`变量声明之前的部分被称为暂时性死区，在暂时性死区中引用`let`变量会报引用错误。

```js
console.log(temp) // Uncaught ReferenceError: temp is not defined

let temp = 'Hi'
```

使用`typeof`访问暂时性死区中的let变量也会报错。

```js
typeof a // 'undefined'
typeof b // 'undefined'
typeof c // Uncaught ReferenceError: c is not defined

var b
let c
```

# const 关键字

与`let`相似，`const`的作用范围是块级作用域，也存在暂时性死区。使用`const`关键字可声明常量，需要注意的是，常量在声明的同时必须进行初始化，而且初始化完成后不能改变。

```js
const a = 2
// 即使是相等的值也不行
a = 2 // Uncaught TypeError: Assignment to constant variable

const b // Uncaught SyntaxError: Missing initializer in const declaration
```

不能改变并不是指该变量指向的值不能改变，而是变量指向的那个内存地址所保存的数据不得改动，对于**原始类型**来说，变量指向的那个内存地址所保存的就是对应的值，而**引用类型**变量指向的内存地址，保存的是指向实际数据的地址，`const`只能保证它保存的地址是不变的，而实际的数据则无法控制（关于原始类型和引用类型的内容之后会讲）。

```js
const person = {}
// 向对象中添加属性
person.age = 24
person.name = 'ikura'

// 尝试改变指向的地址
person = {} // Uncaught TypeError: Assignment to constant variable
```
# 参考资料

**JavaScirpt高级程序设计（第4版）**

**你不知道的JavaScript**

**ECMAScript 6 入门**

**[现代 JavaScript 教程](https://zh.javascript.info/)**

**[MDN Web Docs](https://developer.mozilla.org/zh-CN/)**