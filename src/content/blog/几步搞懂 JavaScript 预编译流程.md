---
# 控制台使用 new Date().toISOString() 生成时间
pubDatetime: 2021-12-14T14:51:00.000Z
modDatetime: 2021-12-14T14:51:00.000Z
title: 几步搞懂 JavaScript 预编译流程
# 页面唯一地址
slug: javascript-pre-build
# 是否在首页精选版块显示此帖子
featured: false
# 将这篇文章标记为草稿。
draft: false
tags:
  - javascript
description: 前言：22届菜鸡前端正在实习中，总感觉基础不牢固，正在补 JavaScript 基础，随笔记录所学、所想。
---

**前言：22届菜鸡前端正在实习中，总感觉基础不牢固，正在补 JavaScript 基础，随笔记录所学、所想。**

## 一、函数体中的预编译过程

**在函数体中搞懂javascript的预编译流程，下面先看代码，打印出执行顺序：**

```javascript
function test(a) {
  console.log(a);
  var a = 1;
  console.log(a);
  function a() {}
  console.log(a);
  var b = function () {};
  console.log(b);
  function c() {}
  console.log(c);
}
test(1);
```

先说答案，依次执行顺序为：

1. `function a() { }`
2. `1`
3. `1`
4. `function () { }`
5. `function c() { }`

为什么？下面解释一下预编译的流程，函数体中只需要4步即可搞懂：

1. 找形参和变量声明
2. 找实参
3. 找函数声明
4. 依次执行
   当页面初次加载时，首先会执行 `test(1)` 函数，然后执行内部操作。

第一步：找每个形式参数：

- 函数体中形参为 `a`，这时形参 `a` 是 `undefined` 所以 `a = undefined` 。
- 函数体中变量有 `a` 和 `b`，变量提升后 `a = undefined`，`b = undefined`。
  第二步：找实参:
- 函数传递的实参为 `1`，这时 `a = 1`。
  第三步：找函数声明：
- 当前 `test` 函数中存在函数体 `function a() { }`，`function c() { }`，函数内部提升函数体，此时 `a = function a() { }`，`c = function c() { }`。
  第四步：自顶向下依次执行：
- 首先执行 `console.log(a)` 打印出 `function a() { }`，然后 `1` 赋值给变量 `a`，此时 `a = 1`；
- 再次执行 `console.log(a)`，此时打印出 `1`，再执行 `console.log(a)` 不变，继续打印出 `1`，然后 `function () { }` 赋值给变量 `b`，此时变量 `b = function () { }`；
- 再次执行 `console.log(b)`，打印出 `function () { }`；
- 最后执行 `console.log(c)`，打印出 `function c() { }`。

## 二、全局预编译过程

**作用于全局作用域中的预编译过程，下面看代码，打印出执行顺序：**

```javascript
var b = 1;
console.log(a);
function a(a) {
  console.log(a);
  var a = 2;
  console.log(a);
  function a() {}
  var b = 5;
  console.log(b);
}
a(1);
```

先说答案，依次执行顺序为：

1. `function a(a){ // ...函数内 }`
2. `function a() { }`
3. `2`
4. `5`

下面详细说一下全局的预编译流程，全局作用域中，只需要3步即可搞懂：

1. 找函数声明
2. 找变量声明
3. 依次执行

第一步：找函数声明

- 找到函数 `function a(a){ // ...函数内 }`，函数提升后此时 `a = function a(a){ // ...函数内 }`；
  第二步：找变量声明：
- 找到变量 `b`，变量提升后此时的 `b = undefined`；
  第三步：自顶向下依次执行：
- 首先将 `1` 赋值给变量 `b`，此时 `b = 1`，执行 `console.log(a)`，打印出 `a = function a(a){ // ...函数内 }`，再次执行函数 `a`，函数体中使用上面的四步操作；
- 找形参和变量声明：找到形参 `a`，此时 `a = undefined`，然后变量`a`，`b`提升，此时`a = undefined`，`b = undefined`；
- 找实参：执行 `a(1)` 时，传递实参 `1`，此时 `a = 1`；
- 找函数体：找到函数 `function a() { }`，此时 `a = function a() { }`；
- 依次执行：执行 `console.log(a)`，打印出 `function a() { }`，然后将 `2` 赋值给变量 `a`，此时 `a = 2`，再次执行 `console.log(a)`，打印出 `2`，再次执行将 `5` 赋值给变量 `b`，此时 `b = 5`，执行到`console.log(b)`时，打印出 `5`。

> 以上就是 JavaScript 中预编译的执行过程，虽然很基础，但是日常开发中用处非常大，建议多看几遍，有些许绕圈的感觉~
> **注意：以下为同一个作用域内**

- 对于同名的变量，`JavaScrip` 会优先采用先声明的变量，后声明的同名变量会**被忽略**，变量默认为 `undefined`。
- 对于同名的函数，`JavaScrip` 会采用最后声明的函数，之前声明的同名函数会**被覆盖**，因为函数声明会指定函数内容，所以保留的是最后一次的声明。
- 对于同名的变量和函数，`JavaScrip` 是函数优先提升，所以同名的变量一定不会被提升，后声明的同名变量会直接**被忽略**。

## 三、关于

- 实习一年半的前端练习生。
- 2022届毕业。
- 目前公司技术栈 React + TypeScript + Egg。
- 啥都做呀，还就那个全干！
- 目前处于啥都会点，啥也都不会。
