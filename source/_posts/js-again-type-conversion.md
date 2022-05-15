---
title: 再学JS之类型转换
date: 2022-05-07 09:26:11
tags: 
  - 类型转换
categories: 
  - 前端
  - JavaScript
cover: https://cdn.jsdelivr.net/gh/ltanfr/ltanfr.github.io/img/js-again.jpeg
---
# 类型转换

JavaScript中类型转换分为显式类型转换和隐式类型转换，顾名思义，显式类型转换指的是直接调用类型转换的方法来进行类型转换，如使用`Number()`、`parseInt()`、`parseFloat()`方法来将其他类型的值转换为`number`类型；隐式类型转换指虽然没有直接使用显示类型转换的方法来进行类型转换，但是根据JS中的规则会自动进行类型转换。从另一个角度来说，只要掌握了类型转换的规则，知道如何触发隐式类型转换，隐式类型转换也可以视作为显式类型转换。

## 类型转换方法（显式类型转换）

常见的类型转换方法有`Number()`、`String()`、`Boolean()`、`parseInt()`、`parseFloat()`、`toString()`。

这些方法分别会尝试将传入的参数转换为`number`、`string`、`boolean`类型的值。

每个方法都有自己的转换规则，其中大部分规则也会应用到隐式类型转换上，少部分方法的转换会有一些差异，如`parseInt()`方法解析字符串`123ads`时会得到结果`123`，而`Number()`方法会得到`NaN`。

## 类型转换规则

### 对象-原始值转换

JS中进行对象转换为原始值时会根据所处的情况即上下文进行推断需要转换为什么类型的原始值，`hint`指的就是推断的结果。

`hint`一共有三种值，分别是`"string"`、`"number"`、`"default"`。当JS无法确定推断上下文情况时会采用`"default"`来处理。

```js
let obj = {x:100}

// hint 为 "string"
alert(obj)

// hint 为 "number"
Number(obj)

// hint 为 "default"
obj == '100'
```

`Symbol.toPrimitive`是一个JS中内置的`Symbol`值，它可以作为对象中的一个默认函数属性存在，当一个对象转换为对应的原始值时，会将`hint`作为参数传入并调用此函数（如果该函数存在的话）。

```js
const obj = {
  [Symbol.toPrimitive](hint) {
		console.log(hint)
    if (hint === "number") {
      return 47;
    }
    if (hint === "string") {
      return "hi";
    }
    return true;
  }
};

alert(obj) // string
+obj // number
obj == '47' // default
```

当`Symbol.toPrimitive`不存在时，JS会寻找对象上的`toString()`方法和`valueOf()`方法来进行转换，方法的调用优先级会根据`hint`的值来判断：

- `"string"`，优先使用`toString()`方法，如果该方法返回的值不是基本值的话，则使用`valueOf()`方法。
- `"number" || "default"`，优先使用`valueOf()`方法，如果该方法返回的值不是原始值，则再使用`toString()`方法。

对象默认的`toString()`方法会返回`"[object Object]"`，而`valueOf()`方法返回对象本身，但JavaScript的许多内置对象都重写了`valueOf()`方法，以实现更适合自身的功能需要。具体可[点击这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf)参考MDN文档。

总结一下，对象转换到原始值主要分为三步：

1. 推断`hint`的值。
2. 如果对象上存在`Symbol.toPrimitive`，则调用该函数处理转换。
3. 如果对象上不存在`Symbol.toPrimitive`，则根据`hint`的值来调用`toString()`或`valueOf()`方法来处理转换。

从之前提到的方法可以看出类型转换最终的结果是`number`、`string`、`boolean`这三种类型之一。知道了对象是如何转换为原始值之后，下面就开始讲解其他类型是如何转换为这三种类型的。

### 转换为 number

**boolean**

- true → 1
- false → 0

**undefined**

- undefined → NaN

**null**

- null → 0

**symbol**

- 转换不了，会抛出异常`Uncaught TypeError: Cannot convert a Symbol value to a number`

**bigInt**

- 字面量形式定义 → 去掉后面的`n`，如`10n → 10`
- BigInt(value) → value（默认`value`能够转换为数值）

**string**

- 数字字符的字符串
    - 十进制、带符号的十进制 → 十进制（注意转换时会忽略数值前的0，如Number(`"0047"`)会返回`47`）
    - 浮点值格式 → 浮点值
    - 十六进制 → 十进制
- 空字符串 → 0
- 以上规则不能解析时 → NaN（注意与方法`parseInt`、`parseFloat`的区别）

**object（可参考对象-原始值转换）**

1. 推断`hint`为`"number"`。
2. 如存在`Symbol.toPrimitive`，则调用该函数处理转换。
3. 如不存在，调用`valueOf()`方法，如果该方法返回的值为原始值的话，则转换结束。
4. 如`valueOf()`方法的返回值不是原始值的话，继续调用`toString()`方法，如果该方法能成功转换为`string`类型的值，则根据`string → number`的规则再进行转换。
5. 如果仍未得到原始值，则抛出`TypeError`异常。

### 转换为 string

**number**

- value → 'value'

**boolean**

- true → 'true'
- false → 'false'

**undefined**

- undefined → 'undefined'

**null**

- null → 'null'

**symbol**

- Symbol(value) → 'Symbol(value)'，如`String(Symbol.iterator)`的返回值为`'Symbol(Symbol.iterator)'`
- Symbol.for(value)→'Symbol(value)'，如`String(Symbol.for('12'))`的返回值为`'Symbol.for('12')'`

**bigInt**

- 字面量形式定义 → 去掉后面的`n`，再转换为字符串，如`10n → '10'`
- BigInt(value) → 'value'（默认`value`能够转换为数值）

**object（可参考对象-原始值转换）**

1. 推断`hint`为`"string"`。
2. 如存在`Symbol.toPrimitive`，则调用该函数处理转换。
3. 如不存在，调用`toString()`方法，如果该方法返回的值为原始值的话，则转换结束。
4. 如`toString()`方法的返回值不是原始值的话，继续调用`valueOf()`方法，如果该方法能成功转换为原始类型的值，则根据前面的规则再进行转换。
5. 如果仍未得到原始值，则抛出`TypeError`异常。

### 转换为 boolean

**number**

- 非零数值 → true
- 0、NaN → false

**string**

- 非空字符串 → true
- 空字符串'' → false

**undefined**

- undefined → false

**null**

- null → false

**symbol**

- Symbol(value) → true
- Symbol.for(value) → true

**bigInt**

- 字面量形式定义，去掉后面的`n`然后按 number → boolean 的规则转换
- BigInt(value)，value → number → boolean

**object**

- 任意对象 → true

## 触发隐式类型转换

在了解了类型转换的规则后，就能清楚地知道显式类型转换的结果，但有时候JS会自动地触发类型转换，这一机制被称为隐式类型转换。一方面它让代码变得晦涩难懂；另一方面，它也能让代码变得更加简洁。

```js
// 显式类型转换
String('1')
// 隐式类型转换
1+''
```

### 触发转换为 number

**一元运算符 +**

```js
+'1' // 1
+'hello' // NaN
+null // 0
+undefined // NaN
+{} // NaN
```

**二元运算符 + - * / % += -=（使用 + 运算符时操作数中不存在字符串）**

```js
// 注意这里的 {} 并不是一个空对象 而是一个代码块
{} + true // 1

// Uncaught TypeError: Cannot mix BigInt and other types, 
// use explicit conversions
10n + 10

// 0 除以任何数都得 0 的规则被打破！
// 与 NaN 的运算结果都会是 NaN
null % undefined // NaN
'' * +{} // NaN

'3.1512' - '0.1' // 3.0511999999999997 （注意这里产生了精度丢失-_—|）
```

当`+`的其中一个操作数是字符串或对象（转换为原始值时得到字符串）时，将会执行字符串拼接。

```js
/**
 * 特殊情况，无法转换为number
 * 推断 hint 为 'default'
 * 数组转换为原始值
 * 首先调用valueOf方法，返回数组本身
 * 再调用toString方法
 * 数组的toString被重写，效果相当于调用join(',')
 * 最后再进行字符串拼接
 */
[1, 2] + [3, 4] // '1,23,4'

/**
 * {} 视为 代码块
 * 原式等同于 +[]
 * 按照一元运算符 + 的规则来
 * 推断 hint 为 'number'
 * 首先调用valueOf方法，返回数组本身
 * 再调用toString方法 返回''
 * '' 转换为 0
 */
{} + [] // 0

// 这里的 {} 是一个空对象
[] + {} // '[object Object]' 
```

**比较运算符（> < <= >=）**

```js
(true > false) // true

// 注意 NaN 的特殊性
(1 < NaN) // false
(1 > NaN) // false
(NaN <= NaN) // false
```

在比较中也存在被称为”抽象关系比较“的特殊情况，其中又分为比较双方都是字符串和存在`object`类型两种情况，如果是第二种情况，需要先将对象转换为原始值，如果结果中出现非字符串，就强制将比较双方转换为数字来比较。

```js
// 同样地，根据语法规则， {} 被视为代码块
({} > []) // Uncaught SyntaxError: Unexpected token '>'

/**
 * 先根据对象 -> 原始值规则进行转换
 * [] 转换为 ''
 * {} 转换为 '[object Object]'
 * 比较双方都是字符串的话按字母顺序来比较
 */
([] < {}) // true
```

此外，还需注意`<=` `>=`的处理方式

```js
let a = { x: 42 }
let b = { y: 43 }
(a<b) // false
(a>b) // false

// a>=b 被处理为 b<a 然后反转结果
(a>=b) // true
// a<=b 被处理为 b>a 然后反转结果
(a<=b) //true**逻辑运算符 ！**
```

### 触发转换为 string

**二元运算符 +（运算元中存在字符串时）**

```js
1 + '0' // '10'
+{} + 'true' // 'NaNtrue'

true + {} // 'true[object Object]'

```

### 触发转换为 boolean

会触发布尔值隐式转换的情况有以下几种：

- if 语句中的条件判断表达式
- for 语句中的条件判断表达式
- while 语句中的条件判断表达式
- 三元表达式（问号表达式）中的条件判断表达式
- ||（或）、&&（与）、!（非）

这里有一个小技巧，使用逻辑运算符`!`可以触发布尔值隐式转换并不准确，因为它还额外做了取反的工作，但是使用`!!`取两次反，就能实现`Boolean()`方法的功能，同时在写法上要比`Boolean()`简洁。

```js
Boolean(1) === !!1 // true
```

逻辑运算符`||`和`&&`，会将左边的操作数作为条件判断表达式来触发布尔值隐式类型转换。换句话说，`||`和`&&`运算符的返回值不一定是布尔值，而是两个操作数中的其中一个，故它们也被称为选择器运算符。

```js
undefined && null // undefined
1 && 2 // 2

0 || 1 // 1
'' || NaN // NaN
```

对于`&&`运算符来说，先对左边的操作数进行布尔值隐式转换，如果结果为假，则返回该操作数；如果结果为真，直接返回右边的操作数。

```js
a && b
// 可以按以下表达式去理解，但两个表达式并不完全相等
a ? b : a
```

对于`||`运算符来说，先对左边的操作数进行布尔值隐式转换，如果结果为假，则返回右边的操作数；如果结果为真，直接返回该操作数。

```js
a || b
// 可以按以下表达式去理解，但两个表达式并不完全相等
a ? a : b
```

在ES5时经常用到这两个运算符来实现一些功能，如使用`||`来实现类似默认参数的功能，使用`&&`来实现类似可选链操作符`?.`的功能。

```js
// ES5 实现默认参数
function debounce(fn, delay) {
  delay = delay || 200;
  let timer = null;
  return function () {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this, arguments);
      timer = null;
    }, delay);
  };
}

// ES6 做法
function debounce(fn, delay = 200) {
  let timer = null;
  return function () {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this, arguments);
      timer = null;
    }, delay);
  };
}

let data = {
	name: 'Joe'
}

// ES5
let name = data && data.name // Joe
// ES6
name = data?.name // Joe
```

## 宽松相等 ==

### 宽松相等 == 与严格相等 ===

在JS中存在宽松相等`==`和严格相等`===`两种等于运算符，它们的区别在于`==`在相等比较过程中当比较双方类型不一致时会先进行类型转换之后再判断；而`===`不会，类型不一致的话直接返回`false`。为了语意清晰，一般会更加推荐使用`===`而不是`==`，但还是有必要了解使用`==`时会发生什么。

> `!=`为宽松不相等，`!==`为严格不相登。
> 

```js
47 == '0x2f' // true
47 === '0x2f' // false
```

### == 的隐式类型转换规则

比较双方类型不一致时，关于`==`的类型转换规则如下：

- 比较双方存在布尔值时，会尝试将布尔值转换为数字再进行比较
- 比较双方其中一个为字符串一个为数字时，会尝试将字符串转换为数字再进行比较。
- 比较双方存在一个为原始值，一个为对象时，会先将对象转换为原始值后再进行比较。

注意关于`==`也存在特殊情况，`null`与`undefined`在宽松比较时可以视为相等，这两种类型在遇到其他类型时一律返回`false`（这种情况下使用`==`要比`===`好一些）。

```js
null == undefined // true
null == null // true
undefined == undefined // true
'0' == null // false
0 == undefined // false
```

### 一些让人头大的宽松比较

`[] == ![]`

看到以上比较可以自己先思考结果，以下给出转换过程

```js
/**
 * [] 为对象子类型 
 * ![] 会进行布尔值隐式转换
 *  根据规则 任意对象转换为布尔值都是true
 *  最后再取反 所以 ![] 的结果为 false 
 * 根据 == 的转换规则
 * ![] 先转换为 false 再转换为数字 0
 * [] 需要转换为原始值
 *  valueOf方法返回本身 不是原始值
 *  调用toString方法 返回 ''
 *  '' 转换为 0
 *  故最后进行的比较为 0 == 0
 */
[] == ![] // true

// 类似地
[47] == 47 // true
[null] == '' // true
'ops' == ['ops'] // true
```

`'true' == true`

同样地，可以自己先思考，以下给出转换过程

```js
/**
 * 比较双方为字符串与布尔值
 *  布尔值转换为数字
 *    true 转换为 1
 *  字符串转换为数字
 *    'true' 转换为 NaN
 * 最后进行的比较是 NaN == 1
 */
'true' == true // false
```

# 常见面试题

1. 实现`a == 1 && a == 2 && a == 3`的结果为`true`（宽松相等发生类型转换，重写`valueOf`方法实现）
    
    > 实现`a === 1 && a === 2 && a === 3`的结果为`true`（严格相等不会发生类型转换，需要借助`Object.defineProperty`来实现数据劫持）
    > 
    
    ```js
    let value = 1;
    Object.defineProperty(window, 'a', {
        get() {
            return value++;
        }
    })
    a === 1 && a === 2 && a === 3 // true
    ```
    
2. `[] == ![]`（宽松相等隐式类型转换规则）
3. `[] + {}`与`{} + []`（隐式类型转换）
    
    > 猜猜 `{} + {}` 的结果
    >
    ```js
    // 这是特殊情况，推测是v8引擎作出的优化
    // chrome 与 nodejs 返回 '[object Object][object Object]'
    // firefox 返回 NaN
    {} + {} // '[object Object][object Object]'
    ```

# 参考资料

**JavaScirpt高级程序设计（第4版）**

**你不知道的JavaScript**

**ECMAScript 6 入门**

**[MDN Web Docs](https://developer.mozilla.org/)**