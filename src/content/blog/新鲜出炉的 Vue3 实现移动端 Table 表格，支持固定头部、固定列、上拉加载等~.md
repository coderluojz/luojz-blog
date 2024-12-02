---
# 控制台使用 new Date().toISOString() 生成时间
pubDatetime: 2024-04-22T08:13:00.000Z
modDatetime: 2024-04-22T08:13:00.000Z
title: 新鲜出炉的 Vue3 实现移动端 Table 表格，支持固定头部、固定列、上拉加载等~
# 页面唯一地址
slug: vue3-table
# 是否在首页精选版块显示此帖子
featured: true
# 将这篇文章标记为草稿。
draft: false
tags:
  - vue
description: 在移动端中可能也需要展示类似PC端的表格功能，找了一些库发现不是能很好的满足业务需求，那只能手搓一个移动端表格了，需要满足固定头、固定列、列排序、单元格超出隐藏点击展开、上拉加载等功能。
---

> 在移动端中可能也需要展示类似PC端的表格功能，找了一些库发现不是能很好的满足业务需求，那只能手搓一个移动端表格了，需要满足固定头、固定列、列排序、单元格超出隐藏点击展开、上拉加载等功能。

**最下面有效果图（gif）**

首先列举一下我们需要的功能：

- 固定头部
- 固定列
- 列排序
- 单元格超出隐藏点击展开
- 上拉加载

下面开始着手实现每个功能，以下代码均使用 `uniapp`、`vue3` 去实现。

### 一、表格参数类型

```typescript
export type SortedType = "ASC" | "DESC" | null;
export interface TableExpose {
  complete: (val: any[]) => void;
  reload: () => void;
  resetPage: (page?: number, pageSize?: number) => void;
}
export interface ColumnsConfig {
  /**
   * 列渲染字段名称
   */
  key: string;
  /**
   * 列标题
   */
  title: string;
  /**
   * 列宽 一个表格一个单元格未定义宽度则自动自适应
   */
  width?: number;
  /**
   * 列排序
   */
  sort?: boolean;
  /**
   * 列固定
   */
  fixed?: boolean;
  /**
   * 列对齐
   */
  align?: "left" | "center" | "right";
  /**
   * 列文本溢出隐藏
   */
  ellipsis?: boolean;
  /**
   * 点击单元格跳转
   */
  jump?: boolean;
}
export interface TableProps {
  /**
   * 表头背景色
   */
  bgHead?: string;
  /**
   * 表内容背景色
   */
  bgBody?: string;

  /**
   * 行唯一id
   */
  rowKey: string;
  /**
   * 内容内边距
   */
  contentPadding?: string;
  /**
   * 默认页码
   */
  defaultPage?: number;
  /**
   * 默认页码大小
   */
  defaultPageSize?: number;
  /**
   * loading加载
   */
  loading?: boolean;
  /**
   * 表头固定
   */
  fixedHeader?: boolean;
  /**
   * 表宽度
   */
  width?: number;
  /**
   * 表高度
   */
  height?: number | string;
  /**
   * 是否显示 => 没有更多数据
   */
  showTextMore?: boolean;
  /**
   * 双向绑定的表数据
   */
  modelValue: any[];
  /**
   * 表头配置
   */
  columns: ColumnsConfig[];
  /**
   * 开启上拉加载
   */
  pullUpLoading?: boolean;
  /**
   * 暂无更多数据提示
   */
  noMoreDataTips?: boolean;
  /**
   * 是否启用排名
   */
  isRank?: boolean;
  /**
   * 是否自动请求数据接口
   */
  isAutoQuery?: boolean;
  /**
   * 数据兜底
   */
  protection?: string;
}
export interface TableEmits {
  /**
   * 排序事件
   */
  (
    e: "sorted",
    {
      columnsKey,
      type,
    }: {
      /**
       * 表列唯一key
       */
      columnsKey: string | null;
      /**
       * 升序、降序类型
       */
      type: SortedType;
    }
  ): void;
  (e: "query", { page, pageSize }: { page: number; pageSize: number }): void;
  (e: "update:modelValue", val: any[]): void;
  (e: "rowClick", row: any, column: ColumnsConfig): void;
}
```

以上是整个表格的所有参数类型。根据自己的业务需求可以额外增减，最基础的表格布局我就不在这儿一一展开了，我们从几个功能点去接入。**涉及到的组件相关的可以自行更改！**

移动端表格需要左右上下滑动最核心的就是使用 `scroll-view` 组件去实现，大部分的框架已经内置了的，比如 `uniapp`、`taro`、`小程序`。如下代码所示：

```html
<scroll-view
  scroll-x
  scroll-y
  enable-flex
  scroll-anchoring
  class="table-container"
  :style="{ height: typeof height === 'string' ? height : height + 'rpx' }"
  :show-scrollbar="false"
>
  <view id="table-content">
    <view class="table-wrap">
      <table-thead>
        <template
          v-for="item in columns"
          :key="item.key"
          #[titleSlot(item)]="scopeInfo"
        >
          <slot
            :name="titleSlot(item)"
            :column="scopeInfo?.column"
            :column-index="scopeInfo?.index"
          ></slot>
        </template>
      </table-thead>
      <table-tbody>
        <template
          v-for="item in columns"
          :key="item.key"
          #[bodySlot(item)]="scopeInfo"
        >
          <slot
            :name="bodySlot(item)"
            :column="scopeInfo?.column"
            :column-index="scopeInfo?.columnIndex"
            :data="scopeInfo?.data"
            :data-index="scopeInfo?.dataIndex"
          ></slot>
        </template>
        <template
          v-for="item in columns"
          :key="item.key"
          #[bodyIconSlot(item)]="scopeInfo"
        >
          <slot
            :name="bodyIconSlot(item)"
            :column="scopeInfo?.column"
            :column-index="scopeInfo?.columnIndex"
            :data="scopeInfo?.data"
            :data-index="scopeInfo?.dataIndex"
          ></slot>
        </template>
      </table-tbody>
    </view>
    <view class="" :style="{ width: maxWidth + 'rpx' }">
      <view
        class="sticky left-0 flex justify-center overflow-hidden"
        :style="{ width: viewportWidth + 'rpx' }"
      >
        <view class="py-32rpx flex w-full flex-col items-center">
          <slot name="empty" v-if="data.length === 0">
            <tm-text class="mt-32rpx" color="#919599" :font-size="24"
              >暂无相关数据</tm-text
            >
          </slot>
        </view>
      </view>
    </view>
  </view>
</scroll-view>
```

### 二、实现固定头部与固定列

表格的固定头部不用多说，其实跟pc是一样的都是通过 `sticky` 去进行一个粘性的定位，使得表格在滚动的时候头部一直固定在上方。如下代码所示：

```html
<view
  class="table-head"
  :style="fixedHeader && { position: 'sticky', top: 0, zIndex: 11 }"
>
  ....表格头部单元格
</view>
```

固定列我们可以根据传递的 `columns` 里面的配置项进行固定几列的操作，假设 `columns` 中第一项里设置了 `fixed: true` 则表示该列需要固定。如下代码所示：

```html
// 表头列固定
<view class="table-tr">
  <view
    class="table-th"
    :class="renderShadow(item, index)"
    :style="[
      renderStyle(item, index),
      {
        backgroundColor: bgHead,
      },
    ]"
    v-for="(item, index) of columns"
    :key="item.key"
  >
    ...单元格内容
  </view>
</view>

// 左侧列固定
<view :class="['table-tr']" v-for="(item, index) of data" :key="item?.[rowKey]">
  <view
    class="table-td"
    :style="[
        renderStyle(c, i),
        {
          backgroundColor: bgBody,
        },
      ]"
    v-for="(c, i) in columns"
    :key="c.key"
  >
    ...单元格内容
  </view>
</view>
```

以上代码中最主要的是 `renderStyle` 的函数实现，拿到传递的对应数据进行列固定的样式赋值，实现方法：

```typescript
const fixedColumnsList = computed(
  () => columns.value?.filter(item => item.fixed) || []
);
const renderStyle = (item: any, index: number): StyleValue => {
  const widthCount = fixedColumnsList.value
    ?.slice(0, index)
    ?.reduce((prev, cur) => {
      return (prev += cur.width);
    }, 0);

  const left = index === 0 ? 0 : widthCount;

  return {
    display: "flex",
    justifyContent:
      item.align === "left"
        ? "flex-start"
        : item.align === "right"
          ? "flex-end"
          : item.align,
    width: item.width ? item.width + "rpx" : "auto",
    position: item.fixed ? "sticky" : "relative",
    left: item.fixed ? left + "rpx" : "auto",
    zIndex: item.fixed ? 10 : 1,
  };
};
```

使用 `fixedColumnsList` 过滤出 `columns` 中 `fixed` 为 `true` 的配置数组，数组中每个对象则为一列，根据过滤出的数组，再进行一个宽度的累加，获取到我们需要粘性定位在表格左侧多少位置比较合适，根据每列对应的`index` 和过滤出来的数据 `fixedColumnsList`计算出来得到 `left` 距离的 `widthCount`，因为我们每一列的 `td` 都是在一个父级里面的，所以需要给每一列的 `td` 设置对应的 `sticky` 和 `left` 距离。

**增加滚动阴影**

```html
// table-head 和 table-body 文件中 // 给 table-th 和 table-td 上增加阴影 // head
<view class="table-th" :class="renderShadow(item, index)"> ...单元格内容 </view>
// body
<view class="table-td" :class="renderShadow(item, index)"> ...单元格内容 </view>
```

表格滚动的阴影应该是在开始滚动的时候才会出现的，所以需要去获取滚动位置。

```typescript
const scrollLeft = ref(0);
const handleScroll = (e: any) => {
  scrollLeft.value = e.detail.scrollLeft;
};
const renderShadow = (item: any, index: number) => {
  if (scrollLeft.value > 0) {
    return (
      item.fixed &&
      index === fixedColumnsList.value?.length - 1 &&
      'fixed-shadow'
    );
  }
};

<style lang="scss">
.fixed-shadow::after {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  width: 30px;
  transform: translateX(100%);
  transition: box-shadow 0.3s;
  content: '';
  pointer-events: none;
  box-shadow: inset 10px 0 8px -8px rgba(5, 5, 5, 0.06);
}
</style>
```

`handleScroll` 方法是在 `scroll-view` 中拿取到的滚动条位置，所以我们要给 `scroll-view` 设置方法

```html
<scroll-view ...属性 @scroll="handleScroll"> ...滚动内容 </scroll-view>
```

### 三、实现列排序

表格每列的排序都是在表头上的，根据点击的表头进行一个正序或倒叙的排序，但是状态应该为三种，在页面中点击表格的时候第一次点击应该为`ASC`正序，第二次点击为`DESC`倒序，第三次点击应该为`null or undefined`，因为第三次需要设置为初始值，所以第三次不应该再进行一个对应的列排序。

```html
<view class="table-tr">
  <view
    class="table-th"
    :class="renderShadow(item, index)"
    :style="[
      renderStyle(item, index),
      {
        backgroundColor: bgHead,
      },
    ]"
    v-for="(item, index) of columns"
    :key="item.key"
  >
    <view @click="handleSort(item.key)">
      ...单元格内容 // 排序图标
      <view v-if="item.sort">
        <tm-icon
          class="sort-icon up"
          name="tmicon-sort-up"
          :font-size="22"
          :color="renderSortIconColor(item.key, 'ASC')"
        ></tm-icon>
        <tm-icon
          class="sort-icon down"
          name="tmicon-sort-down"
          :font-size="22"
          :color="renderSortIconColor(item.key, 'DESC')"
        ></tm-icon>
      </view>
    </view>
  </view>
</view>
```

列排序也是根据 `columns` 中的配置项 `sort` 决定的，下面实现 `handleSort` 和 `renderSortIconColor` 方法。

```typescript
const sortInfo = ref<{
  key: string | null;
  type: SortedType;
}>({
  key: null,
  type: null,
});
const handleSort = (key: string) => {
  if (sortInfo.value?.key !== key) {
    sortInfo.value.key = key;
    sortInfo.value.type = "ASC";
  } else if (sortInfo.value.type === "ASC") {
    sortInfo.value.type = "DESC";
  } else if (sortInfo.value.type === "DESC") {
    sortInfo.value.key = null;
    sortInfo.value.type = null;
  }
  emits("sorted", {
    columnsKey: key,
    type: sortInfo.value.type,
  });
};

const renderSortIconColor = (key: string, type: SortedType) => {
  return sortInfo.value?.key === key && sortInfo.value?.type === type
    ? "#00B35D"
    : "#777979";
};
```

上面根据点击表头的事件进行三种状态的切换，并且改变点击的 `icon` 图标，最后通过 `emits('sorted', {columnsKey: key, type: sortInfo.value.type })` 把每一次点击的事件抛了出去，由调用者去请求接口进行排序调用。

### 四、实现单元格超出隐藏点击展开

单元格超出隐藏可以直接用文本省略去做，点击操作也是类似，但是我们的 `UI` 在省略文本的旁边加了一个 `icon` 用来表示当前文本是否隐藏，区分点击事件，这就又要写很多代码了...

将内容单元格作为一个新的组件去封装

```html
<view
  class="cell-item"
  :style="{
      justifyContent:
        column.align === 'left'
          ? 'flex-start'
          : column.align === 'right'
            ? 'flex-end'
            : column.align,
    }"
  @click="handleShowMore"
>
  <text
    :id="`cell-${column.key}-${item?.[rowKey]}`"
    :class="[
        classShowMore(column.ellipsis, column.key, item?.[rowKey]),
      ]"
    style="line-break: anywhere"
    :style="{
        textAlign: column.align,
      }"
    >{{ item[column.key] ?? protection }}</text
  >
  <text class="copy" :id="`copy-cell-${column.key}-${item?.[rowKey]}`"
    >{{ item[column.key] }}</text
  >
  <view class="arrow-icon" v-if="overflow">
    <tm-icon
      color="#ABAEB3"
      :font-size="24"
      name="tmicon-angle-down"
      v-if="!!classShowMore(column.ellipsis, column.key, item?.[rowKey])"
      class="ml-4rpx"
    ></tm-icon>
    <tm-icon
      color="#ABAEB3"
      :font-size="24"
      name="tmicon-angle-up"
      v-else
    ></tm-icon>
  </view>
</view>
```

```typescript
const Instance = getCurrentInstance();

const overflow = ref(false);

const classShowMore = (
  ellipsis: boolean,
  key: string | number,
  rowKey: string | number
) => {
  if (!ellipsis || !overflow.value) return "";
  if (props?.moreInfo?.key !== key || rowKey !== props?.moreInfo?.rowKey) {
    return "cell-text-overflow";
  }
  if (props?.moreInfo?.key === key && props?.moreInfo?.rowKey === rowKey) {
    return props?.moreInfo?.show ? "" : "cell-text-overflow";
  }
};

const handleShowMore = () => {
  const { ellipsis, key } = props?.column || {};
  emits("show-more", {
    ellipsis,
    key,
    rowKey: props?.item?.[rowKey],
    overflow: overflow.value,
  });
};

const checkOverflow = cellId => {
  const query = uni.createSelectorQuery().in(Instance.proxy);
  query.select(`#${cellId}`).boundingClientRect();
  query.select(`#copy-${cellId}`).boundingClientRect();
  query.exec(([element, copyElement]) => {
    overflow.value = copyElement?.width > element?.width;
  });
};

onMounted(() => {
  if (props?.column?.ellipsis) {
    nextTick(() => {
      checkOverflow(`cell-${props?.column?.key}-${props?.item?.[rowKey]}`);
    });
  }
});
```

```css
<style scoped lang="scss">
  .cell-item {
    display: flex;
    align-items: center;
    box-sizing: border-box;
    .arrow-icon {
      margin-left: 4rpx;
    }
    .copy {
      position: fixed;
      top: -9999px;
      left: -9999px;
      display: block;
      white-space: nowrap;
      visibility: hidden;
    }
  }
  .cell-text-overflow {
    display: -webkit-box;
    white-space: inherit;
    overflow: hidden;
    text-overflow: ellipsis;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
  }
</style>
```

以上是子组件的内容，进入页面通过`dom`查找对应的单元格然后进行复制文本内容与当前单元格的内容进行一个宽度的对比，假设`复制文本的内容 > 当前单元格文本的内容 * 2（超出2行隐藏）`的情况下则表示省略文本，并且展示对应的 `icon`。点击单元格的时候抛出 `emits('show-more')` 方法，由父组件进行控制，这样每个单元格是互斥的效果。

```typescript
// table-body.vue
const showMore = ref({
  key: null,
  rowKey: null,
  show: false,
});

const handleShowMore = ({
  ellipsis,
  key,
  rowKey,
  overflow,
}: ShowMoreParams) => {
  if (!ellipsis || !overflow) return "";
  if (key !== showMore.value?.key || rowKey !== showMore.value?.rowKey) {
    showMore.value.key = key;
    showMore.value.rowKey = rowKey;
    showMore.value.show = true;
  } else if (key === showMore.value?.key && rowKey === showMore.value?.rowKey) {
    showMore.value.show = false;
    showMore.value.key = null;
    showMore.value.rowKey = null;
  }
};
```

### 五、实现上拉加载

上拉加载也是通过 `scroll-view` 中的事件 `@scrolltolower="handleScrollTolower"` 去实现的。

```typescript
const handleScrollTolower = debounce(async (e: BaseEvent) => {
  if (e.detail.direction !== "bottom" || !props?.pullUpLoading) return;
  if (props?.noMoreDataTips && maxMorePage.value) {
    uni.showToast({
      title: "暂无更多数据",
      icon: "none",
    });
    return;
  }
  if (props.loading) {
    return;
  }
  Object.assign(pageInfo, { page: pageInfo.page + 1 });
  emits("query", { page: pageInfo.page, pageSize: pageInfo.pageSize });
}, 300);
```

获取到下拉事件，然后判断是否有更多数据，进行下一次的页码增加，并且抛出当前页码与页面大小让调用者去调用接口方法。

### 六、总结

以上是一个很简单的移动端表格实现，但是也包含了一些常见的功能，类似固定列固定头部这种功能，其实也可以通过 多个 `scroll-view` 互相控制滚动位置去实现，这样就不会导致 `sticky` 在移动端展示可能出现抖动的问题。

具体的代码这里就不放了 有点儿太多了 贴出来不太友好... 贴个效果图...

![CPT2404221642-342x605.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3beb94672dec4dce8136eb72d3bf42fa~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=342&h=605&s=6932215&e=gif&f=161&b=faf4f3)
