---
title: Rxjs
tags: js
---

# Rxjs
-------
## 函数式编程
首先学习 Rxjs，我们先了解两种较为传统的编程思想：`面向对象编程思想`、`面向过程过程编`程思想；

**面向对象编程**：要求实例化对象具有封装、继承、多态等变现形式。其优势是属性黑盒内置，外层调用不能直接修改内部的属性，只能通过对外暴露的方法。

**面向过程编程**：目前具有代表性的是`函数式编程`，`响应式编程`。其优势是数据在函数之间传递，形成的数据流，纯函数引用后即被回收，运行时占用内存较少。

## Rxjs 是什么?
通过对第一章的学习了解，首先 Rxjs 采用的是面向过程的编程思想，数据流在函数间流动，所以函数可以有效传递数据流，结合观察者模式 + 迭代器模式来管控。

## Observable & Observer

Observable 是一个可以被订阅的数据集合。Observer是一个订阅者。订阅者在Rxjs中充当一个被动接收数据的觉得。一旦订阅了，数据集合就会给订阅者传输数据。订阅的操作交给 `subscribe`。

``` javascript
import {Observable} from 'rxjs/Observable';
const onSubscribe = observer => {
    console.log(observer, "---1---");
    observer.next(1);
    observer.next(2);
    observer.next(3);
};
const source$ = new Observable(onSubscribe);
console.log(source$, "---2---");
const theObserver = {
        next: item => console.log(item)
} 
source$.subscribe(theObserver);
```
![](https://i.loli.net/2021/05/27/BJRx23Wvi4Cdo8y.jpg)

![](https://i.loli.net/2021/05/27/RNfOV2EJBvgcZ4s.jpg)

数据流不可能都是一种从A到B的流向。过程中我们可能对数据做很多的中间处理。可以利用 `pipe` 方法将数据流不断的分流，如下如所示:

![](https://i.loli.net/2021/05/26/dgWFzZQ7ScTE3xm.jpg)

## Hot Observable & Cold Observable
当一个 Observable 的数据如果同时被两个订阅者订阅，并且两个订阅的订阅时机有先后，B 在 A 订阅之后在订阅。B 在订阅过程中的数据，是否考虑之前流入A的那些数据呢?

- 考虑： Cold Observable
- 不考虑：Hot Observable

## 弹珠图
地址：[弹珠图](https://rxviz.com/)
弹珠图存在意义是考虑复杂场景下，帮助解决开发者更好的理解数据流的流向问题。

## 操作符
![image-20211227175307524](https://tva1.sinaimg.cn/large/008i3skNly1gxsjh093hnj30ej0fu0t6.jpg)

Rxjs 中操作符的种类有很多，但是无外乎上面这些类。



