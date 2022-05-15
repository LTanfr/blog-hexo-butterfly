---
title: 初见 RxJS
date: 2022-05-06 15:10:16
updated:
tags: 
  - RxJS
categories: 
  - 前端
  - RxJS
keywords:
description:
top_img:
cover: https://cdn.jsdelivr.net/gh/ltanfr/ltanfr.github.io/img/rxjs-first-sight.jpeg
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
# 初识 RxJS

# 写在前面

刚入职了新公司，负责维护一个原来使用`Angular`构建的项目，奈何自己一直使用的是`React`。不过既来之，则安之。顺便插一句，熟悉`Vue`的朋友应该会觉得很亲切，其中很多语法如`v-if`、`v-bind`，与`Angualr`中的`ngIf`、`ngFor`的用法都如出一辙。

看了两周左右的`Angular`，说一下第一印象，`Angular`确实是一个非常齐全的框架，它不像`Vue`跟`React`需要自己去考虑其他东西，如状态管理方案、网路请求方案等，而在`Angular`中就不需要考虑这些，它已经替你做好了选择，开发者只需要学会它里面的那一套东西，然后专注于业务开发即可。
其中`RxJS`就是`Anuglar`提供的异步编程方案。

`RxJS`官网描述自己是`Reactive Extensions Library for JavaScript（为JS而打造的响应式可扩展编程）`，也可以把`RxJS`当做处理异步行为的`Lodash`库`（Think of RxJS as Lodash for events）`。`RxJS`是编程范式函数式编程(`Functional Programming`)与响应式编程(`Reactive Programming`)的结合，换句话说，`RxJS`是一种编程思想，使用它能够让编写的代码更优雅，可读性更高。或许正是这样，在国内知名度并不是很高，网上可参考的资料也不是很多，自己学习这部分时主要是参考了`J.H. Blog`的`30天精通RxJS`系列文章，该系列文章是2016年到2017年由作者洪名辰编写，虽然其中很多的API已经被官方废弃，但文章的编程思想仍未过时，学习时结合RxJS官方文档，便很快能够上手。


# 什么是Observable
## Observer Pattern

观察者模式在程序设计中并不少见，如`DOM`操作中的通过给`DOM`元素绑定相应的事件去监听该`DOM`元素的相应行为（如鼠标点击、滑动等），并在该行为触发时执行对应的事件；或是大名鼎鼎的`Redux`的实现，其都是以观察者模式的思想为核心。

```js
// redux 的简单实现
export const createStore = (reducer) => {
  // 公共状态
  let currentState = {};
  // 监听者队列
  let listeners = [];
  // getter
  function getState() {
    return currentState;
  }
  // setter
  function dispatch(action) {
    currentState = reducer(currentState, action);
		// 当dispatch触发，即状态更新时通知listeners
    listeners.forEach((fn) => fn());
  }
  // 添加监听事件
  function subscribe(fn) {
    listeners.push(fn);
  }
  // 初始化store数据
  dispatch({ type: "@@REDUX_INIT" });
  return { getState, dispatch, subscribe };
};
```

## Iterator Pattern

`iterator`（迭代器）的概念实际上也并不陌生，ES6中也加入了原生的iterator接口，如`Array`、`Map`、`Set`、`String`等JS中内置的数据结构都实现了该接口。实现`iterator`接口的方法是为其添加`Symbol.iterator`属性，其中包含`next`方法，该方法返回的迭代器对象包含两个属性：`value`和`done`。`done`指的是遍历是否结束，`value`为下一次迭代的值，`done`为`true`时`value`一般为`undefined`。

```js
const iterableExample = {
	[Symbol.iterator]: function() {
		return {
			next: function() {
				return {
					value: 'Sunday',
					done: true
				}
			}
		}
	}
}

const arr = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']
const iterator = arr[Symbol.iterator]()
iterator.next() // {value: 'Monday', done: false}
iterator.next() // {value: 'Tuesday', done: false}
iterator.next() // {value: 'Wednesday', done: false}
```

迭代器无须了解与其关联的可迭代对象的结构，只需要知道如何取得连续的值。这样就很好地分离了`iterator`与`iterable`的概念。

## Observable

`Observable`结合了`Observer`与`Iterator`的思想，像`Observe`与`Iterator`一样以流的形式包装数据，同时能够以`Observe`的方式通知更新，以`Iterator`的方式去处理数据。这样说起来可能有点抽象，但是结合`Observable`实例就能很好地理解。

# 创建 Observable

创建`Observable`实例的基本方法,使用`Observable`构造函数来创建一个`Observable`实例对象，其中可以传入一个回调函数,接收一个`subscriber`，用来指定该如何处理数据。需要注意的是，`Observable`作为被观察的对象，需要使用`subscribe`方法来通知`subscriber`去对`Observable`作出处理。

```js
import { Observable } from 'rxjs';

let observalbe1 = new Observable((subscriber) => {
  subscriber.next('Suda');
  subscriber.next('Nana');
});

// 使用subscrible方法来订阅Observable
observalbe1.subscribe(console.log);
// Suda
// Nana

// 处理非同步行为
let observalbe2 = new Observable((subscriber) => {
  subscriber.next('Suda');
  setTimeout(() => {
    subscriber.next('Kasumi');
  }, 1000);
  subscriber.next('Nana');
});

observalbe2.subscribe(console.log);
// Suda
// Nana
// Kasumi

```

订阅`Observable`的被称为观察者，观察者具有三种行为（方法）来处理`Observable`的数据：

- `next`：当Observable发送新的值，即调用其中的next方法时，观察者的`next`方法就会被调用。
- `complete`：在`Observable`中完成遍历即调用其中的`complete`方法时，观察者的`complete`方法会被调用，在此之后的`next`方法不会执行。
- `error`：在`Observable`中出现错误时，观察者的`error`方法会被调用。

```js
let observable3 = new Observable((subscriber) => {
  subscriber.next('Inuyasha');
  subscriber.next('Kikyo');
  subscriber.complete();
  subscriber.next('Kagome');
});

// 与Promise类似的处理方法
let observer = {
  next: function (value) {
    console.log(value);
  },
  error: function (error) {
    console.log(error);
  },
  complete: function () {
    console.log('complete');
  },
};

observable3.subscribe(observer);
// Inuyasha
// Kikyo
// complete
```

需要注意的是，只传入一个函数时会被默认指定为`next`的回调函数。

# 创建 Observable 实例的常用方法

## of

> `of<T>(...args: (SchedulerLike | T)[]): Observable<T>`
> 

需要注意其参数与方法`Function.prototype.call`类似，需要为一连串的参数，而不直接是一个数组，处理的顺序为传入参数的顺序，下图来自rxjs官网。

![of.png](https://rxjs.dev/assets/images/marble-diagrams/of.png)

```js
import { of } from 'rxjs';

const source = of(1, 2, 3);
source.subscribe({
  next: (value) => {
    console.log(value);
  },
});
```

## from

> `from<T>(input: ObservableInput<T>, scheduler?: SchedulerLike): Observable<T>`
> 

该方法弥补了`of`方法不能直接传入数组等数据结构等缺陷，参数为`Array`、`类数组对象`、`Promise`、`可迭代对象`或`类 Observable 对象`

```js
import { from } from 'rxjs';

const source = from([1, 2, 3]);
source.subscribe({
  next: (value) => {
    console.log(value);
  },
});
```

## interval

> `interval(period: number = 0, scheduler: SchedulerLike = asyncScheduler): Observable<number>`
> 

这个方法能够实现JS中`setInterval`的功能，需要注意第一个参数是以`ms`为单位的时间间隔。

![图片来自rxjs官网](https://rxjs.dev/assets/images/marble-diagrams/interval.png)

```js
import { interval } from 'rxjs';

const numbers = interval(2000);
// 以2秒的间隔从0开始在控制台输出数字
numbers.subscribe((n) => console.log(n));
```

## Timer

> `timer(dueTime: number | Date = 0, intervalOrScheduler?: number | SchedulerLike, scheduler: SchedulerLike = asyncScheduler): Observable<number>`
> 

该方法与`Interval`类似，第一个参数指定间隔多少时间后调用，可以是数字，也可以是具体日期，第二个参数指定了第一次调用之后每次调用的时间间隔。

```js
import { timer } from 'rxjs';

// 只传一个参数表示指需要调用一次
const source1 = timer(1000);
source1.subscribe({
  next: (value) => console.log(value),
  complete: () => console.log('complete'),
  error: (error) => console.log(error),
});
// 0
// complete

// 第一次调用后会每隔1秒调用1次
const source2 = timer(2000, 1000);
source2.subscribe({
  next: (value) => console.log(value),
  complete: () => console.log('complete'),
  error: (error) => console.log(error),
});
// 2秒之后开始输出 0 1 2 3 4 ...

```

## EMPTY, NEVER

这两个`Observable`常量分别用来替代在以前版本的`empty`方法与`never`方法，前者会直接`complete`，后者不会被执行。

```js
import { EMPTY, NEVER } from 'rxjs';

EMPTY.subscribe({
  next: (value) => console.log(value),
  complete: () => console.log('complete'),
  error: (error) => console.log(error),
});
// complete

// 不会被执行
NEVER.subscribe(() => console.log('never be called'));
```

## throwError

> `throwError(errorOrErrorFactory: any, scheduler?: SchedulerLike): Observable<never>`
> 

该方法会直接抛出异常，与`Promise.reject`方法类似。

```js
import { throwError } from 'rxjs';

throwError(() => console.log('Ops!')).subscribe();
```

## **Subscription**

在对`Observable`进行`subscribe`之后，如果不进行释放，会一直占用资源，从而造成内存泄漏。为此，就跟`setTimeout`、`setInterval`等方法一样，需要我们在使用完后手动进行释放。

```js
import { interval } from 'rxjs';

const source = interval(1000);
const subscriptipn = source.subscribe();

// 使用完后手动释放
subscriptipn.unsubscribe();
```

# 参考资料

**[30天精通RxJS](https://blog.jerry-hong.com/series/rxjs)**

**[RxJS官方文档](https://rxjs.dev/)**

**ECMAScript 6 入门**

**[MDN Web Docs](https://developer.mozilla.org/zh-CN/)**