---
title: Vue小记-初识
date: 2017-03-24 17:00:00
tags:
 - Vue
 - 笔记

categories:
 - Vue

---

Vue的初识笔记，语法什么的见[https://cn.vuejs.org/v2/guide/syntax.html]

<!-- MORE -->
## 什么是Vue  
- 渐进式框架
- 受 [MVVM模型](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)启发
- 操作 [虚拟DOM] (https://github.com/livoras/blog/issues/13)


## 什么是虚拟DOM
单个元素就已经很大，DOM很慢。  
如果要修改某处，直接操作整个DOM可行，但会造成很多无用的性能消耗。  
DOM可以用JavaScript对象表示出来。

虚拟DOM 就是用户只操作 js 数据，由框架来实现最终的DOM **精准** 渲染。
