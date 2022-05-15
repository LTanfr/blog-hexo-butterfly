---
title: 再学 JS 之数据类型 —— number
date: 2022-05-05 16:35:21
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
## 原始类型(primitive values)

原始类型一共有七种：`number`、`string`、`boolean`、`undefined`、`null`、`bigInt`、`symbol`。

原始类型的值本身无法改变，对一个变量重新赋值的时候该原始的值并不会变。

> 在设计原始类型时，设计者想尽可能地保持原始类型的轻量，但同时考虑到使用者会对原始类型做出很多操作，需要为对原始类型的操作封装成方法。这样的想法显然十分矛盾，而最后的解决方案是为原始类型提供了特殊的“对象包装器”（`String`、`Number`、`Boolean`、`Symbol`），其中封装了许多对原始类型操作的方法，使用完其中的方法后创建的临时对象就会被销毁。

```js
// 不能为原始类型的实例直接添加属性
let str = 'Hi'
str.test = 4
console.log(str.test) // undefined

// 可以显式地为原始类型进行“包装”
let num = new Number(2)
num.test = 33
console.log(num.test) // 33

// 在调用原始类型的方法时，由于原始类型上并没有方法
// 会创建一个包含字符串字面值的特殊对象，该对象具有此方法
// 调用完此方法后会将该临时对象销毁，留下原始值
const lower = 'lowercase'
const upper = lower.toUpperCase()
console.log(lower) // lowercase
console.log(upper) // LOWERCASE
```

# number

在JavaScript中数字是基于IEEE 754 标准的双精度 64 位二进制格式（IEEE754可视化，可以帮助理解）进行存储，能够安全表示的范围为$-(2^{53}-1)$（即-9007199254740991）到$2^{53}-1$，超过此范围的数字可用BigInt类型来存储。

```js
typeof 1 === 'number'

// NaN(Not-a-Number) 表示非数值
typeof NaN === 'number'

Number.MAX_SAFE_INTEGER // 9007199254740991

Number.MIN_SAFE_INTEGER // -9007199254740991

// Infinity 表示无穷
typeof Infinity === 'number'

// +Infinity 表示正无穷 -Infinity 表示负无穷
+Infinity > 0 // true
```

需要注意的是，在 JavaScript 中，可以直接使用`number`类型表示浮点数，但需要注意的是，存储浮点数使用的内存空间是存储整数值的两倍（[具体原理涉及汇编，有基础的话可参考这篇文章](https://zhuanlan.zhihu.com/p/352723802)），所以在小数点后面没有数字或者小数点后跟的是 0（如 23.00），会被转换为整数。

> 实际业务场景经常会有需求需要格式化数字，如保留小数后两位，以防像 23.00 这样的数字被处理为 23，可以使用`toFixed`方法对数字显式地进行格式化

```js
const floatNum1 = 1.9

const floatNum2 = .2 // 小数点前可以不需要整数，但这种做法并不推荐

const floatNum3 = 3.000 // 会被当作整数3处理
```

## 精度丢失问题（0.1+0.2 !== 0.3）

造成这个原因的是在 JavaScript 中数字以二进制的形式存储在内存中，而像 0.1、0.2 这样的小数在进行[二进制](https://zh.wikipedia.org/wiki/%E4%BA%8C%E8%BF%9B%E5%88%B6)转换后会出现无限循环，而在 JavaScript 中数字以 64 位格式[IEEE 754](https://zh.wikipedia.org/wiki/IEEE_754) 表示：其中 52 位被用于存储这些数字，其中 11 位用于存储小数点的位置（对于整数，它们为零），而 1 位用于符号。超过其范围就会将数字舍入到最接近的可能数字，从而造成了极小的精度损失。

> 虽然存在一位用于表示符号，但是`+0`与`-0`在 JavaScript 中被认为是相等的，但可以使用`Object.is()`方法可以判断`+0`与`-0`不相等，此为特殊情况。
>
> ```js
> +0 === -0 // true
> Object.is(+0, -0) // false
> ```

```js
0.1+0.2 // 0.30000000000000004

// 该数字转换为二进制后存在53位，超出范围
9999999999999999 // 10000000000000000
```

并不只是 JavaScript 存在该问题，其他使用 IEEE 754 标准来存储数字的语言也存在该问题。

## 使用科学计数法表示

```js
// 使用科学计数法
1e2 === 100 // true
1e-3 === 0.001 // true
```

## 二进制，八进制与十六进制

javaScript 中默认支持这三种进制的写法，其他进制可以通过方法`parseInt`转换

```js
// 不正确的进制表示会报语法错误
// 二进制数字每位只能是1或0
0b103 // Uncaught SyntaxError: Invalid or unexpected token

// 不同进制的数字在进行数学操作时都会被视作为十进制数字
0b101+017 === 20 // true
0xff + 0o377 === 510 // true

// 二进制 使用0b前缀
0b101 === 5 // true

// 八进制 使用0o前缀
0o17 === 15 // true

// 非严格模式下可以使用前缀0来表示八进制（不推荐）
017 === 15 // true
// 超过八进制每位的表示范围（0-7）会忽略前缀0，被视为十进制数字
019 === 19 // true

// 十六进制 使用0x前缀
0xff === 255 // true
// 大小写不会造成影响
0xfF === 0xff // true
```

## 常用方法

> 不能直接在整数数字后面采用点（.）加上方法名的形式来调用方法，因为 JavaScript 语法中隐含了第一个点之后的部分为小数部分，所以直接调用会报语法错误。可以使用两个点(只对十进制数字整数有效)或者用括号将数字部分包裹的形式来进行调用。如果数字本身存在小数部分则可以直接采用点加上方法名的形式来调用方法。

```js
6.toString() // Uncaught SyntaxError: Invalid or unexpected token

(0xfe).toString() === '254' // true
(20).toString() === '20' // true
20..toString() === '20' // true

```

### **num.toString(base)**

将数字转换为字符串，参数`base`表示需要转换的进制，默认值为 10。

```js
// 十进制转换为二进制
2.toString(2) === '10' // true

// 八进制转换为十六进制
(0o377).toString(16) === 'ff' // true
```

### **num.toFixed(digits)**

格式化数字，返回字符串。参数`digits`表示需要保留几位小数。该方法会对数字进行四舍五入。

```js
// 保留小数点后3位
1.23.toFixed(3) === '1.230' // true

// 保留小数点后2位
1.256.toFixed(2) === '1.26' // true
```

注意 toFixed 方法同样存在精度丢失的问题。

```js
// 如果小数部分是一个无限的二进制,存储会造成精度损失

1.35.toFixed(20) // 1.35000000000000008882
1.55.toFixed(1) === '1.6' // true

6.35.toFixed(20) // 6.34999999999999964473
6.35.toFixed(1) === '6.3' // true

// 解决方法为先将其转换为能够在二进制数字系统中可以被精确地表示的数字
// 等操作完后再进行还原
(6.35 * 10).toFixed(20) // 63.50000000000000000000
```

### **isNaN(value)**

判断一个值是否为 NaN

> 注意`NaN === NaN`的比较结果为`false`，这是因为`NaN`是独一无二的，它不等于任何东西，这也是`isNaN`方法存在的意义。不过，使用`Object.is()`方法同样可以判断是否为`NaN`。

```js
isNaN(12) === false // true

// 此处发生了类型的隐式转换
// 如果参数不是数字，会尝试将参数转换为数值
isNaN('11') === false // true

isNaN('ab') === true // true

// 使用Object.is()方法判断
Object.is(NaN, NaN) // true
```

### **isFinite(value)**

判断传入的数字是否为有限数字，参数是  `NaN`、正无穷大或者负无穷大，会返回`false`，其他返回  `true`。

### **parseInt(string, radix)**

### **parseFloat(string, radix)**

将字符串强制转换为十进制数字。参数`string`表示需要解析的字符串，`radix`表示解析时采用什么进制，有效范围为`2到36`。

不传第二个参数或传入的参数值为 0 时默认按照十进制进行解析，传入其他非有效进制的值时会返回`NaN`。

第一个参数中数字的进制如果与第二个参数中的进制不匹配时也会返回`NaN`。

```js
// 如果第一个字符不能转换为数字，则直接返回NaN
// 遇到其他进制的数字会正常解析，如二进制以0b开头等
parseInt('a12') // NaN
parseInt('0b101') === 5 // true

// 直到读取到非数字为止，会转换之前读取到的数字
parseInt('2a1') === 2 // true

// 第二个参数传入非有效进制
parseFloat('1.2', 1) // NaN

// 第二个参数传入0时按照十进制解析
parseFloat('1.2', 0) === 1.2 // true

// 3 并非有效的二进制表示
parseInt('3', 2) // NaN
```

### **Number(value)**

`number`类型的对象包装器，可以通过`Number()`构造函数来创建`number`类型，这个函数的参数`value`可以为其他类型如`stirng`、`boolean`等类型的值。由此可见，该函数主要用于数值转换，根据参数，具体的转换规则如下：

- `boolean`，`true`返回`1`，`false`返回`0`。
- `number`，直接返回值。
- `null`，返回`0`。
- `undefined`，返回`NaN`。
- `string`，规则如下:
  - 字符串为数字时（包括二进制、十六进制、浮点数等能被 JavaScript 失败的数字）返回对应的十进制数字
  - 空字符串返回`0`
  - 如果字符串中包含其他字符，则直接返回`NaN`，注意就算是像`'12a'`这样的值也会返回`NaN`，这点与`parseInt`方法中解析规则直到读取到非数字为止不同。

这里已经涉及到**隐式类型转换**，具体的会在之后详细地解释。

## 常见面试题

1. `0.1 + 0.2 !== 0.3`的原因（考察 JavaScript 中数字的存储）
2. `['1','2','3'].map(parseInt)`的输出结果（考察`map`、`parseInt`方法以及函数式编程中管道的概念）

## 参考资料

**JavaScirpt高级程序设计（第4版）**

****[现代 JavaScript 教程](https://zh.javascript.info/)****

**[MDN Web Docs](https://developer.mozilla.org/zh-CN/)**

