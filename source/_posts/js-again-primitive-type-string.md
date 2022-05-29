---
title: 再学 JS 之数据类型 —— string
date: 2022-05-05 19:20:53
updated:
tags: 
	- 数据类型
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
# string

在JavaScript中字符串必须被括号包裹，括号包括`''`、`""`、````。其中````为ES6中引入的符号，又被称为模板字符串。

```js
typeof 'hello' === 'string'
typeof "true" === 'string'
typeof '' === 'string'
typeof `aa` === 'string'
```

## 单引与双引

单引号和双引号的作用一样，但是要前后保持一致，不能混用。另外单引号和双引号都不允许字符串跨行，但是可以使用换行符`\n`来实现换行，与其他语言一样，可以通过转义字符\来表示特殊字符，如`\\`、`\'`等。

```js
let str = 'hello
		world' // Uncaught SyntaxError: Invalid or unexpected token
str = '\\aa'
console.log(str) // \aa
```

在表示长字符串时，一行可能表示不下，可以使用+运算符来连接多个字符串，或者在每一行字符串的末尾使用\来表示下一行继续。

```js
const str1 = 'Love\'s not Time\'s fool, though rosy lips and cheeks' + 
						 'within his bending sickle\'s compass come;' + 
						 'Love alters not with his brief hours and weeks,' +
						 'But bears it out even to the edge of doom.'

const str2 = 'In faith I do not love thee with mine eyes,\
For they in thee a thousand errors note;'
```

## 模板字符串

在ES6引入的模板字符串可以视作增强版的字符串，用` `` `包裹。比起普通的字符串，它可以直接定义跨行字符串，而且可以直接在字符串中以`${...}`嵌入JavaScript表达式或变量。

```js
const name = 'Tim';
let str = `hello, ${name}`
console.log(str) // hello, Tim

const template
```

## 不可变值

需要注意的是，string属于原始类型，其值不可变。

```js
let str = 'hello';
console.log(str) // hello
// 可以用str.length来访问字符串的长度
// 用str[pos]的方式来获取第pos个字符
str[0] = 'aaaa' // 无效赋值
console.log(str) // hello
```

## 常用方法

### str.indexOf(subStr[, pos])

该方法指的是从`str`的`pos`位置开始查找`subStr`，如果能够找到，则返回`subStr`第一次出现的位置；如果未能找到，则返回`-1`。其中第二个参数为可选参数，如果不写的话默认从`0`开始，如果`pos`小于`0`也会从`0`开始查找，大于`str.length`的话会直接返回`-1`。

```js
let str = 'hello world'
console.log(str.indexOf('world', 2)) // 6
console.log(str.indexOf('a')) // -1
console.log(str.indexOf('hello', 1)) // -1
console.log(str.indexOf('h', 20)) // -1
```

另外存在一个现象，当`subStr`的值为`''`时，如果第二个参数`pos`的值在`0`到`str.length`的范围内时，返回值为`pos`，小于`0`返回`0`，大于`str.length`返回`str.length`。

```js
let str = 'test'
console.log(str.indexOf('', -2)) // 0
console.log(str.indexOf('', 3)) // 3
console.log(str.indexOf('', 8)) // 4
```

### str.lastIndexOf(substr[, pos])

与`indexOf`方法类似，只不过该方法查找的是最后一次`substr`出现的位置，同时是从`pos`开始从后往前查找。

```js
// pos只限制待匹配字符串的开头
console.log('ababab'.lastIndexOf('ab',2)) // 2
```

### str.charAt(pos)

该方法会返回`str`中位置为`pos`的字符。如果`pos`的值超出了`0`到`str.length-1`的范围则会返回`''`。如果未传入`pos`的话，则默认`pos`为`0`。

```js
console.log('Snow Falling on Cedars'.charAt(5)) // F
```

### str.substring(start[, end])

该方法在str中截取从`start`的位置到`end`的位置（不包含`end`）的字符。其中第二个参数为可选参数，未传入的话将截取从`start`到`str.length-1`的字符。

如果`start`的值大于`end`，则截取的字符串为从`end`至`start`（不包含`start`）。

其中任一参数的值小于`0`或大于`str.length`的话，会被当作`0`或者`str.length`处理。任一参数的值为`NaN`（包括隐式转换后为`NaN`）的话也会被当作`0`处理。

```js
let str = 'The Snows of Kilimanjaro'

console.log(str.substring(3,0)) // The
console.log(str.substring({},3)) // The
console.log(str.substring(-2,25)) // The Snows of Kilimanjaro
```

> MDN文档中提到`substr`方法在将来有可能被移除掉，因为`substring`方法功能类似，所以尽量使用`substring`方法去替代它。
> 

### str.toUpperCase()

### str.toLowerCase()

这两个方法顾名思义，将字符串转换为大写/小写。

```js
let str = 'abCdEfG'
console.log(str.toUpperCase()) // ABCDEFG
console.log(str.toLowerCase()) // abcdefg
```

### str.includes(substr[, pos])

该方法是ES6新加的方法，用来判断`substr`是否包含在`str`中，是的话返回`true`，否则返回`false`。其中`pos`为可选参数，指的是开始查找的位置，未传入的话从`0`开始。

```js
let str = 'Kagome To Inuyasha'
console.log(str.includes('Kikyo')) // false
```

### str.startsWith(substr[, pos])

### str.endsWith(substr[,pos])

这两个也是ES6新加的方法，顾名思义，判断`str`是否以`substr`开头/结尾，是的话返回`true`，否则返回`false`。可选参数`includes`方法中的可选参数是同样的作用，但要注意在`endsWith`中查找的方式与`lastIndexOf`一致。

```js
let str = 'We Made a Beautiful Bouquet'
console.log(str.startsWith('We')) // true
console.log(str.endsWith('auti', str.length-10)) // false
```

# 参考资料

**JavaScirpt高级程序设计（第4版）**

****[现代 JavaScript 教程](https://zh.javascript.info/)****

**[MDN Web Docs](https://developer.mozilla.org/zh-CN/)**