---
# 控制台使用 new Date().toISOString() 生成时间
pubDatetime: 2023-02-27T08:24:00.000Z
modDatetime: 2023-02-27T08:24:00.000Z
title: Pinia？看这篇就够了！
# 页面唯一地址
slug: vue3-pinia
# 是否在首页精选版块显示此帖子
featured: false
# 将这篇文章标记为草稿。
draft: false
tags:
  - vue3
description: Pinia（发音为/piːnjʌ/，如英语中的“peenya”）是最接近 piña（西班牙语中的菠萝）的词。 Pinia本质上依然是一个全局状态管理的库，用于跨组件、页面进行状态共享（这点和Vuex、Redux、Mobx一样）。
---

> Pinia（发音为`/piːnjʌ/`，如英语中的“**peenya**”）是最接近 **piña**（西班牙语中的菠萝）的词。 Pinia本质上依然是一个**全局状态管理的库**，用于**跨组件、页面进行状态共享**（这点和Vuex、Redux、Mobx一样）。

## 一、Pinia 和 Vuex 的区别

- Mutations 不再存在，不需要再多次操作进行数据提交，不用再写冗长的代码。
- 提供了一个更简单的 API，具有更少的规范，提供了 Composition-API 风格的 API，并且在使用 TypeScript 的时候有更好的类型推断支持。
- 不用动态添加 Store，默认情况下已经是动态的了。但是仍然可以随时手动使用 Store 进行注册，但因为它是自动的，就不用担心是否注册。
- 不再有命名空间的概念，不用记住它们的复杂关系。
- 不再有 Modules 的嵌套结构，我们可以灵活使用每一个 Store，因为它们是通过扁平化的方式来相互使用的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b9d0acf3d4b40c49e1eb0af5d653bea~tplv-k3u1fbpfcp-zoom-1.image)

## 二、简单使用 Pinia

- 首先，进行安装 `Pinia`

```
// 使用 npm 安装
npm install pinia
// 使用 yarn 安装
yarn add pinia
// 使用 pnpm 安装
pnpm add pinia
```

- 创建一个 `pinia` 并且注册至全局

Store 文件

```
// scr/stores/index.ts
import { createPinia } from "pinia";

const MyPinia = createPinia();

export default MyPinia;
```

注册全局

```
// src/main.ts
import { createApp } from "vue";

import App from "./App.vue";
import router from "./router";

import "./assets/main.css";

import MyPinia from "./stores/index";

const app = createApp(App);

// 全局注册 pinia
app.use(MyPinia);
app.use(router);
app.mount("#app");
```

- 通过 `defineStore` 定义一个名为 `counter` 的`Store`，并且我们使用 `use` 开头命名一个函数`useCounterStore`进行导出。

```
// scr/stores/counter.ts
import { ref, computed } from "vue";
import { defineStore } from "pinia";

export const useCounterStore = defineStore("counter", () => {
  const count = ref(0);
  const doubleCount = computed(() => count.value * 2);

  const increment = () => {
    count.value++;
  };
  const decrement = () => {
    count.value--;
  };

  return { count, doubleCount, increment, decrement };
});
```

- 在 `App.vue` 中直接引入 `useCounterStore` 进行调用

```
// src/App.vue
<script setup lang="ts">
import { useCounterStore } from "@/stores/counter";
// 注意：这里导入的函数，不能被解构，那么会失去响应式。
// 例如不能这样： const { count, increment, decrement, doubleCount } = useCounterStore();
const counterStore = useCounterStore();
</script>

<template>
  <div class="app-wrapper">
    <div>count：{{ counterStore.count }}</div>
    <div>doubleCount：{{ counterStore.doubleCount }}</div>
    <div>
      <button @click="counterStore.increment()">增加</button>
      <button @click="counterStore.decrement()">减少</button>
    </div>
  </div>
</template>

<style scoped></style>
```

- 注意导入的函数，不能被解构，那么会失去响应式。如果非要使用则如下：

```
<script setup lang="ts">
import { useCounterStore } from "@/stores/counter";
import { toRefs } from "vue";
// or
// import { storeToRefs } from "pinia";
const counterStore = useCounterStore();
const { count, doubleCount, increment, decrement } = toRefs(counterStore);
</script>

<template>
  <div class="app-wrapper">
    <div>count：{{ count }}</div>
    <div>doubleCount：{{ doubleCount }}</div>
    <div>
      <button @click="increment()">增加</button>
      <button @click="decrement()">减少</button>
    </div>
  </div>
</template>

<style scoped></style>
```

> - 以上我们全局注册 `Store` 后，通过 `defineStore`定义了一个名为 `counter`的Store，`name` 为 `counter`，也称为 `id`，是必须的，`Pinia` 使用它来将 `Store` 链接到 `devtools`。
> - 返回的函数我们使用 `use`开头的小驼峰格式方案命名，这是约定的规范。

Pinia 在浏览器中的调试的工具 `vue-devtools `，在 Chrome 或 Edge 的扩展中直接搜索 vue-devtools 进行安装即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/265fd1198fa34e5eaa39f5a24f730fc7~tplv-k3u1fbpfcp-zoom-1.image)

## 三、操作 State

- 读取和写入 state：

默认情况下，可以通过 store 实例访问状态来直接进行读取和写入状态；

```
const counterStore = useCounterStore();
const add = () => {
  counterStore.count++;
};
```

可以直接在页面中定义函数调用，也可以写在定义 counter 的文件当中。

- 重置 state：

```
const counterStore = useCounterStore();
const resetStore = () => {
  // 调用 $reset 函数进行数据重置为初始值
  counterStore.$reset();
};
```

- 改变 state：

```
const counterStore = useCounterStore();
const add = () => {
  // 调用 $patch 同时应用多个修改
  counterStore.$patch({
    count: 10,
    // ...可同时修改多个属性
  });
};
```

- 替换 state：

```
const counterStore = useCounterStore();
const replaceState = () => {
  // 通过 $state 直接赋值替换整个对象
  counterStore.$state = {
    count: 100,
  };
};
```

## 四、认识和定义Getters

- 使用对象的方式创建一个新的 `userStore`

```
// scr/stores/user.ts
import { defineStore } from "pinia";
export const useUserStore = defineStore("user", {
  state: () => ({
    name: "ll",
    age: 18,
    count: 100,
  }),
  getters: {
    doubleCount(state) {
      return state.count * 2;
    },
    doubleCountAddOne(): number {
      return this.doubleCount + 1;
    },
  },
});

// scr/views/UserView.vue
<script setup lang="ts">
import { useUserStore } from "@/stores/user";
const userStore = useUserStore();
</script>

<template>
  <div class="user-wrapper">
    <h2>name：{{ userStore.name }}</h2>
    <h2>age：{{ userStore.age }}</h2>
    <h2>count：{{ userStore.count }}</h2>
    <h2>doubleCount：{{ userStore.doubleCount }}</h2>
    <h2>doubleCountAddOne：{{ userStore.doubleCountAddOne }}</h2>
  </div>
</template>

<style scoped>
.user-wrapper h2 {
  font-size: 24px;
  font-weight: bold;
}
</style>
```

渲染效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afffa74d74f54f68a1aa90b57afdde32~tplv-k3u1fbpfcp-zoom-1.image)

> 访问 **getter** 中定义的 `doubleCount` ，并且可以访问其他的 **getter** `doubleCountAddOne`

- 将参数传递给 getter

```
// scr/stores/user.ts
state: () => ({
  userList: [
    {
      id: 1,
      name: "ll",
    },
    {
      id: 2,
      name: "zz",
    },
  ],
}),
getters: {
  getUserById(state) {
    return (id: number) => state.userList.find((item) => item.id === id);
  },
}
// scr/views/UserView.vue
<h2>id1：{{ userStore.getUserById(1) }}</h2>
<h2>id2：{{ userStore.getUserById(2) }}</h2>
```

渲染效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82b42e2bd5c348e6ab260b62593c23a6~tplv-k3u1fbpfcp-zoom-1.image)

- 访问其他 Store 中的 getter

```
import { useCounterStore } from "@/stores/counter";
otherGetter(state) {
  const counterStore = useCounterStore();
  return state.count + counterStore.count;
}
```

## 五、认识和定义Actions

- Actions 相当于组件中的 methods。 它可以使用 `defineStore()` 中的 `actions` 属性定义，并且非常适合定义业务逻辑：

```
// scr/stores/home.ts
import { defineStore } from "pinia";
export const useHomeStore = defineStore("home", {
  state: () => ({
    count: 0,
  }),
  actions: {
    increment() {
      this.count++;
    },
    randomCount() {
      this.count = Math.round(Math.random() * 100);
    },
    setData(val: number) {
      this.count = val;
    },
  },
});
```

```
<!-- scr/views/HomeView.vue-->
<script setup lang="ts">
import { useHomeStore } from "@/stores/home";
const homeStore = useHomeStore();
</script>

<template>
  <div class="home-wrapper">
    <h2>count：{{ homeStore.count }}</h2>
    <div>
      <button @click="homeStore.increment">增加</button>
      <button @click="homeStore.randomCount">随机</button>
      <button @click="homeStore.setData(666)">设置</button>
    </div>
  </div>
</template>

<style scoped>
.home-wrapper h2 {
  font-size: 24px;
  font-weight: bold;
}
</style>
```

> 以上通过 Actions 定义的方法操作视图更新，并且可以通过调用函数传递参数来直接设置数据

- 与 getter 的操作一样，但是不同的是 actions 是支持异步的，你可以在其中使用 await 任何 API 调用，如下所示，使用 fetch 进行一个异步操作：

```
export const useHomeStore = defineStore("home", {
  state: () => ({
    banners: [],
  }),
  actions: {
    async fetchHomeBanner() {
      const res = await fetch("http://123.207.32.32:8000/home/multidata");
      const data = await res.json();
      this.banners = data.data.banner.list;
    },
  },
});
```

```
<script setup lang="ts">
import { useHomeStore } from "@/stores/home";
const homeStore = useHomeStore();
</script>

<template>
  <div class="home-wrapper">
    <h2>banners：</h2>
    <ul>
      <template v-for="item in homeStore.banners" :key="item.id">
        <li>{{ item.title }}</li>
      </template>
    </ul>
    <div>
      <button @click="homeStore.fetchHomeBanner">获取数据</button>
    </div>
  </div>
</template>

<style scoped>
.home-wrapper h2 {
  font-size: 24px;
  font-weight: bold;
}
</style>
```

> 你可以完全自由地设置你想要的任何参数并返回任何数据。 调用 Action 时，一切都会自动推断！Actions 像 methods 一样被调用。

## 六、总结

Pinia 相较于 Vuex 来说，更简单的API，更少的写法，并且对于TypeScript 支持非常好，可以自动推断类型，可以随意调用，不用局限于 mutations，大大提高了开发效率，并且降低了学习成本。
