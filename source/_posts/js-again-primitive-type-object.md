---
title: 再学 JS 之数据类型 —— object
date: 2022-05-06 19:46:33
tags: 
  - 数据类型
categories: 
	- 前端
  - JavaScript
cover: https://cdn.jsdelivr.net/gh/ltanfr/ltanfr.github.io/img/js-again.jpeg
---
# object

在JavaScript中，可以认为除了原始类型以外，一切皆为对象。像函数、数组等都是对象的子类型，只是表现形式很像其他语言中单独的类型。所以`typeof`在判断`object`类型的变量时只会返回`object`或`function`，无法做到精确地判断具体是某一子类型。判断具体是哪一子类型时一般会使用`instanceof`来判断，但当页面存在多个`iframe`的时候，它也会失灵。最稳妥的方法是使用`Object.prototype.toString.call(obj)`去判断，这里就不做过多地展开。同样地，由于涉及`object`可以延伸的内容非常的多，如`原型`、`原型链`、`this指向`、`扩展运算符`等，这里只对`object`作一些基础的介绍。

## 创建对象

创建一个对象有两种方式，一是通过构造函数来创建，二是通过字面量创建。

```js
const obj1 = new Object(); // 通过构造函数创建
const obj2 = {}; // 通过字面量创建
```

在创建对象的时候可以向其中预置一些属性，也可以在创建后之后添加属性或对属性做出修改。

属性是以键值对的形式存储在对象中，属性的值可以是任意的类型。注意要删除属性的时候需要用关键字`delete`。访问属性可以用`object.key`或是`object['key']`的形式。

```js
const user = {
	name: 'Joe',
  age: '20',
	introduce: function() {
		console.log(`name: ${this.name}, age: ${this.age}`)
	}
}

user.introduce() // name: Joe, age: 20
user.age = '19'
user.introduce() // name: Joe, age: 19

delete user.age
user.introduce() // name: Joe, age: undefined
```

## 引用类型与原始类型的区别

在`JavaScript`中，只有原始类型与引用类型，而这两种类型最大的不同在于存储方式上的选择。就像谷歌开发的`V8`引擎在处理垃圾回收时会区别对待占用空间大小不同、存活时间长短不同的对象一样，`JavaScript`对此也作出了抉择。

一般来说，原始值就是最简单的数据，占用的空间较小，使用的频率高；而引用值则是由多个值构成的对象，占用的空间较大，使用的频率相对较低。基于这样的特性通常我们也将原始类型称为简单类型，将引用类型称为复杂类型。

原始类型的值会直接保存在栈内存中，访问原始类型变量的方式是按值访问，也就是说，我们访问到的就是存储在变量中实际的值。而访问引用类型的变量时我们实际上访问到的是对该对象的引用，而并非对象本身，即访问引用类型变量的方式是引用访问。

这样设计的原因就是考虑到了原始类型使用频率高、占用内存空间小的特性，就算直接将值放在内存中也不会影响程序运行时的性能，而且访问起来更快。引用类型往往是由多个值构成，因为使用频率没原始类型那么高，同时占用内存较大，如果直接将值放在内存中会造成程序运行时性能下降。

再深入一点，`JavaScript`中内存空间分为栈与堆，先不讨论栈与堆在数据结构上的特性，只需知道变量都是放在栈中，对于原始类型来说，存放的就是实际的值；对于引用类型来说，存放的是指向实际对象值的地址。

```js
let a1 = 1
let a2 = true
let a3 = '#fff'
let a4 = [{x:100}]
let a5 = null
```

存储方式如下图所示

![原始类型与引用类型.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d55eb02b-bd83-4430-8b75-ad5131d5f41c/原始类型与引用类型.png)

## 浅拷贝与深拷贝

使用赋值运算符（`=`）能够直接对变量进行拷贝，包括原始类型的变量与引用类型的变量，但需要注意的是，这里拷贝的是栈中的值。也就是说，在使用赋值运算符时，拷贝原始类型的变量，获取到的时原始类型实际的值；拷贝引用类型的值，获取到的是**指向引用类型实际值的地址**。

对于原始类型来说，因为可以直接获取到实际的值，并不存在所谓的浅拷贝与深拷贝；对于引用类型来说，**浅拷贝**就是直接通过赋值运算符去拷贝引用地址，而不是真实的值；**深拷贝**指的才是完全拷贝了一份引用类型变量的值。

### 浅拷贝

由于通过浅拷贝获取到的是同一个引用地址，所以使用相等运算符（`==`）和严格相等运算符（`===`）来与拷贝前的变量比较时是相等的。

```js
let a1 = {x:100}
let a2 = a1
a2 == a1 // true
a2 === a1 // true
```

同时，通过浅拷贝获取的变量在修改原来的对象时，修改的是存在堆中的实际值，而存在栈中的引用地址并没有改变。也就是说，浅拷贝前的变量与浅拷贝后的变量实际指向的对象是同一个，无论哪一个变量去修改对象，另一个变量也能感知到变化。

```js
let obj1 = { x:100 }
let obj2 = obj1
obj2.x = 'yahaha'

obj1 == obj2 // true
obj1 === obj2 // true

obj1.x // 'yahaha'
```

### 深拷贝

首先明确一点，深拷贝是在堆中另外开辟一块空间来存放拷贝对象的值，其引用地址不一样，是无法使用相等运算符与严格相等运算符来判断两个对象是否真正相等。

深拷贝无法简单地使用赋值运算符来完成，这里讲一个简易的实现深拷贝的思路：

1. 判断传入参数为原始类型还是引用类型，如果是原始类型直接返回；如果是引用类型则继续进行下一步。

```js
// 暂时忽略 typeof value === 'function' 的情况
const isObject = (value) => typeof value === 'object' && value !== null
```

2. 判断引用类型具体是哪一子类型，根据子类型的特性来采取不同的拷贝方式。

```js
let result

// 这里以数组和对象举例
// 用 Object.prototype.toString.call(value) 可判断对象子类型
const isArray = (value) => Array.isArray(value)
result = isArray(value) ? [] : {}
```

3. 通过递归的方式来拷贝其对象属性。

```js
// 数组和对象都可以用下列方式来进行递归拷贝
for(let key in value) {
	// 确保属性来自 value 本身而不是原型链
	// Object.property.hasOwnProperty() 方法可判断传入的键是否来自对象实例本身
	if(value.hasOwnProperty(key)) {
			// 递归拷贝
			result[key] = deepClone(value[key])
	}
}
```

当然上述内容有很多不足，如没有考虑循环引用、每一种对象子类型、对象子类型是否可遍历等问题，光是这些细节就可以拿出来单独讲很久，这里只提供一个整体思路。完整的深拷贝学习可以参考`lodash`源码中`deepClone`部分。

```js
const isObject = (value) => typeof value === 'object' && value !== null;
const isArray = (value) => Array.isArray(value);

const deepClone = (value) => {
  if (!isObject(value)) {
    return value;
  }

  let result = isArray(value) ? [] : {};

  for (const key in value) {
    if (value.hasOwnProperty(key)) {
      result[key] = deepClone(value[key]);
    }
  }
  return result;
};
```

## 对象比较

前面提到对象之间无法通过相等运算符或者严格相等运算符来进行比较。这里也提供一个大体的思路以供参考，同样地，完整的对象比较可以参考`lodash`源码的`eqDeep`部分。

```js
const isObj = (obj) => typeof obj === 'object' && obj !== null;
function isEqual(obj1, obj2) {
  // 传入基本类型时直接进行比较
  if (!isObj(obj1) || !isObj(obj2)) {
    return obj1 === obj2;
  }
  // 判断传入的是否为同一对象
  if (obj1 === obj2) {
    return true;
  }
  // 判断对象上的key的数量是否一致
	// Object.keys() 方法返回传入对象上所有键组成的数组
  const obj1Keys = Object.keys(obj1);
  const obj2Keys = Object.keys(obj2);
  if (obj1Keys !== obj2Keys) {
    return false;
  }
  // 递归比较对象上的键值是否相等
  for (let key in obj1) {
    const res = isEqual(obj1[key], obj2[key]);
    if (!res) {
      return false;
    }
  }
  return true;
}
```

## for...in

这里单独提一下`for...in`语句，因为这个语句是为了遍历对象属性而生的，需要注意的是该语句在遍历对象属性时，每一次遍历获取的是对象的键，而不是值。`for...in`语句同样也可以用于遍历数组，但是并不推荐。`for...in`不应该用于迭代一个关注索引顺序的 `Array`。

## 方法

对象中的方法分为实例方法与静态方法，其中方法非常多，常用的实例方法有`Object.prototype.hasOwnProperty()`、`Object.prototype.toString()`、`Object.prototype.valueOf()`等，常用的静态方法有`Object.defineProperty()`、`Object.assign()`、`Object.keys()`等，许多方法的使用都是建立在对对象有一定了解的基础之上，会在之后结合具体场景一个个慢慢提到，这里不详细描述。

# 常见面试题

下列哪种访问对象属性的方式是不正确的（对象属性的访问方式）

```js
const bird = {
  size: "small"
};

const mouse = {
  name: "Mickey",
  small: true
};
```

- A: `mouse.bird.size`
- B: `mouse[bird.size]`
- C: `mouse[bird["size"]]`
- D: All of them are valid

代码输出结果(引用类型的存储方式、浅拷贝与深拷贝)

```js
let obj1 = {a: 100}
let obj2 = obj1
obj1.a = 200
console.log(obj2.a)
```

代码输出结果(包装对象、类型转换)

```js
const a = 100;
const b = new Number(100)
console.log(a==b)
console.log(a===b)
```

下列代码是否会报错(对象子类型)

```js
function test() {
	console.log('test!')
}

test.a = 100
```

手写深拷贝、对象比较

# 参考资料

**JavaScirpt高级程序设计（第4版）**

**你不知道的JavaScript**

**ECMAScript 6 入门**

**MDN Web Docs(https://developer.mozilla.org/)**

**lodash(https://lodash.com/)**

