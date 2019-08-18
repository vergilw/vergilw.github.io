---
layout: post
title: VueJS中props和data的区别
date: 2019-08-18
---

### props和data

props和data都可以用来存储JS对象，修改它们都会触发重新渲染。
从字面上理解，props是对象的属性，属性是不可变，如果改变了就不是原来的对象了，而是另一个的对象。data是对象的数据，借用其他响应式框架的说法应该是状态，状态是可变的，例如时钟对象的时针任意变化指向不同的值，它始终都是原来的那个时钟。

这实际上引出了响应式框架里一个重要概念，就是无状态组件stateless和有状态组件stateful。

### 区别

|   | props | data | 
| --- | --- | --- |
| Can get initial value from parent Component? | Yes | Yes |
| Can be changed by parent Component? | Yes | No |
| Can set default values inside Component? | Yes | Yes |
| Can change inside Component? | No | Yes |
| Can set initial value for child Components? | Yes | Yes |
| Can change in child Components? | Yes | No |

### 选择

使用data是可选的，但是由于使用data增加了组件的复杂性，因此优先选择无状态的组件。我们应该避免使用太多有状态组件。
具体选择可以参考[Stateful与Stateless的选择](https://vergilw.github.io/2019/04/flutter-ui-widget-pattern/)

### Reference
[props vs state](https://github.com/uberVU/react-guide/blob/master/props-vs-state.md)
