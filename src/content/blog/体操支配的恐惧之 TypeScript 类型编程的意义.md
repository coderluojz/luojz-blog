---
# 控制台使用 new Date().toISOString() 生成时间
pubDatetime: 2022-12-28T15:29:00.000Z
modDatetime: 2022-12-28T15:29:00.000Z
title: 体操支配的恐惧之 TypeScript 类型编程的意义
# 页面唯一地址
slug: typescript-code
# 是否在首页精选版块显示此帖子
featured: false
# 将这篇文章标记为草稿。
draft: false
tags:
  - typescript
description: 在 ts 基础的学习中，我们通常会对一些数据做各种类型声明，比如对象类型、数组类型、函数类型等，但是如果我们需要动态生成一些类型，或者对类型做一些变化该如何去做呢？
---

## 摘要

在 ts 基础的学习中，我们通常会对一些数据做各种类型声明，比如对象类型、数组类型、函数类型等，但是如果我们需要动态生成一些类型，或者对类型做一些变化该如何去做呢？

类型编程中，我们尽可能的写出具体的类型可能会更精确的提示出这个变量是什么类型，这样类型提示的精准度高了，在开发中的提交将会更好。

这就是类型编程的意义：**需要动态生成类型的场景，必然要用类型编程做一些运算。有的场景下可以不用类型编程，但是用了能够有更精准的类型提示和检查。**

下面我将通过举例来说明：

## ParseQueryString

`parseQueryString` 是一个格式化浏览器参数的方法，可以将浏览器的 `get` 参数转换成 对象的格式，下面先通过 JS 函数实现一个基本的功能：

```javascript
const parseQueryString = queryStr => {
  if (!queryStr) {
    return {};
  }
  const queryArr = queryStr.split("&");
  const result = queryArr.reduce((prev, cur) => {
    const [key, value] = cur.split("=");
    if (prev[key]) {
      if (Array.isArray(prev[key])) {
        !prev[key].includes(value) && prev[key].push(value);
      } else {
        prev[key] = [prev[key], value];
      }
    } else {
      prev[key] = value;
    }
    return prev;
  }, {});
  return result;
};
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3441c44ca85e4441b29f81e63ae02485~tplv-k3u1fbpfcp-watermark.image?)

以上就是一个基本的转换方法。如果要给这个函数加上一个类型，应该怎么去加呢？

大部分来说应该会这样去加：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff31603b82cc4376930fd380b767080b~tplv-k3u1fbpfcp-watermark.image?)

`Record` 是一个 TypeScript 内置的高级类型，会通过映射的方式去生成对象类型：

```
type Record<K extends keyof any, T> = {
    [P in K]: T;
}
```

比如传入 `'a' | 'b'` 作为 `key`， 任意数作为 `value`，就可以生成这样的对象类型：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c19e8e9248f42db9ccbd04f75b8ad7d~tplv-k3u1fbpfcp-watermark.image?)

而 `keyof any` 其实是 `string | number | symbol` 的联合类型

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecec083237a945b58338b0adefe54cc3~tplv-k3u1fbpfcp-watermark.image?)

但是我们这样使用推导的类型会存在一个问题：**返回的对象不能提示出有哪些属性：**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b5456e7cdea4243add41170aac15999~tplv-k3u1fbpfcp-watermark.image?)
那怎么才能让这个返回的对象能有提示呢？

这就需要用到类型编程了，我们可以写一个 `ParseQueryString` 类型：

`a=1&b=2&c=3&a=2&a=1`，这样的字符串个数明显是一个不确定的值，遇到数量不确定的问题，我们可以直接通过递归来解决：

- 首先，需要递归提取 `&` 分割的字符串：

```typescript
type ParseQueryString<Str extends string> =
  Str extends `${infer Param}&${infer Rest}`
    ? MergeParams<ParseParam<Param>, ParseQueryString<Rest>>
    : ParseParam<Str>;
```

类型参数 `Str` 作为待处理的字符串，通过 `extends` 约束为 `string` 类型。

提取 `&` 分割的字符串到 `infer` 声明的局部变量 `Param` 里，后面剩余的字符串就放到 `Rest` 里。

通过 `ParseParam` 类型来处理分割后单个的字符串，例如 `a=1`，剩下的字符串 `Rest` 通过递归继续处理，然后把这些处理的结果合并在一起，也就是类型 `MergeParams`。

当提取不到 `&` 分割的字符串时，递归就结束了，把剩下的字符串也使用类型 `ParseParam` 来处理。

`ParseParam` 的实现就是提取和构造，将 `a=1` 提取出来构造成一个对象 `{ a: 1}`：

```typescript
type ParseParam<Param extends string> =
  Param extends `${infer Key}=${infer Value}`
    ? {
        [K in Key]: Value;
      }
    : {};
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14d6a590d4964e0faf278582cf578792~tplv-k3u1fbpfcp-watermark.image?)

- 在使用类型 `ParseParam` 进行提取构造处理完成后，最后就需要把这些构造出来的对象类型合并成一个就好了：

```typescript
type MergeParmas<
  OneParam extends Record<string, any>,
  OtherParam extends Record<string, any>,
> = {
  [Key in keyof OneParam | keyof OtherParam]: Key extends keyof OneParam
    ? Key extends keyof OtherParam
      ? MergeValues<OneParam[Key], OtherParam[Key]>
      : OneParam[Key]
    : Key extends keyof OtherParam
      ? OtherParam[Key]
      : never;
};
```

类型参数 `OneParam、OtherParam` 是要合并的字符串对象，约束为对象类型，`key` 为 `string`，值为任意类型。

构造一个新的对象类型进行返回，对象来自两个的合并，也就是 `Key in keyof OneParam | keyof OtherParam`。

值也需要合并：
如果两个对象类型中都有，那就合并成一个，也就是 `MergeValues<OneParams[Key], OtherParam[Key]>`。

否则，如果是 `OneParam` 中的，就取 `OneParam[Key]`，如果是 `OtherParam` 中的，那么就取 `OtherParam[Key]`。

- 类型 `MergeValues` 的合并逻辑就是如果两个值是同一个就返回一个，否则构造出一个数组类型来合并：

```typescript
type MergeValues<One, Other> = One extends Other
  ? One
  : Other extends unknown[]
    ? [One, ...Other]
    : [One, Other];
```

类型参数 `One、Other` 是要合并的两个值。

如果两者是同一个类型，也就是 `One extends Other`，就返回任意一个。

否则，如果是数组，就做数组合并，否则构造一个数组将两个类型放进行。

下面测试合并的参数：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bbe13f1088b7486f82dde94794087284~tplv-k3u1fbpfcp-watermark.image?)

每个字符串的解析和构造对象类型，多个对象类型的合并实现过后，合并起来也就实现了字符串解析：

```typescript
type ParseQueryString<Str extends string> =
  Str extends `${infer Param}&${infer Rest}`
    ? MergeParams<ParseParam<Param>, ParseQueryString<Rest>>
    : ParseParam<Str>;
```

- 最后我们给函数加上类型：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08d21c0891654f10883b7dc65650e286~tplv-k3u1fbpfcp-watermark.image?)

这里我们通过函数重载的方式来声明类型，不然返回值可能和 `ParseQueryString` 的返回值类型匹配不上。

这样，我们就实现出了一个比较精准的类型提示，这就是类型体操的意义。

全部代码：

```typescript
type ParseParam<Param extends string> =
  Param extends `${infer Key}=${infer Value}`
    ? {
        [K in Key]: Value;
      }
    : Record<string, any>;

type MergeValues<One, Other> = One extends Other
  ? One
  : Other extends unknown[]
    ? [One, ...Other]
    : [One, Other];

type MergeParams<
  OneParam extends Record<string, any>,
  OtherParam extends Record<string, any>,
> = {
  [Key in keyof OneParam | keyof OtherParam]: Key extends keyof OneParam
    ? Key extends keyof OtherParam
      ? MergeValues<OneParam[Key], OtherParam[Key]>
      : OneParam[Key]
    : Key extends keyof OtherParam
      ? OtherParam[Key]
      : never;
};

type ParseQueryString<Str extends string> =
  Str extends `${infer Param}&${infer Rest}`
    ? MergeParams<ParseParam<Param>, ParseQueryString<Rest>>
    : ParseParam<Str>;

type IObj = Record<keyof any, any>;

function parseQueryString<Str extends string>(
  queryStr: Str
): ParseQueryString<Str>;

function parseQueryString(queryStr: string) {
  if (!queryStr) {
    return {};
  }
  const queryArr = queryStr.split("&");
  const result = queryArr.reduce((prev: IObj, cur: string) => {
    const [key, value] = cur.split("=");
    if (prev[key]) {
      if (Array.isArray(prev[key])) {
        prev[key].push(value);
      } else {
        prev[key] = [prev[key], value];
      }
    } else {
      prev[key] = value;
    }
    return prev;
  }, {});
  return result;
}

const res = parseQueryString("a=1&b=2&c=3&a=2");
```

对比一下使用类型编程和不使用类型编程的体验：
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b5456e7cdea4243add41170aac15999~tplv-k3u1fbpfcp-watermark.image?)

vs

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aa6e78220004f3d8ff8d0615af3a8a3~tplv-k3u1fbpfcp-watermark.image?)

类型体操的意义之一：实现更精准的类型提示和检查。

以上是神光大佬的小册[TypeScript 类型体操通关秘籍](https://juejin.cn/book/7047524421182947366/section)理解之后的阅读记录。
