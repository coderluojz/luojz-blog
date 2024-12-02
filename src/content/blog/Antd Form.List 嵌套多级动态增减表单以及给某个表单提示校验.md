---
# 控制台使用 new Date().toISOString() 生成时间
pubDatetime: 2023-05-17T12:01:00.000Z
modDatetime: 2023-05-17T12:01:00.000Z
title: Antd Form.List 嵌套多级动态增减表单以及给某个表单提示校验
# 页面唯一地址
slug: antd-form-list
# 是否在首页精选版块显示此帖子
featured: true
# 将这篇文章标记为草稿。
draft: false
tags:
  - react
  - antd
description: 由于公司业务需求，需要用到动态的增减表单，然后就想起了使用 `Antd` 的 `Form.List` 去完成这个功能。
---

## 一、前言

由于公司业务需求，需要用到动态的增减表单，然后就想起了使用 `Antd` 的 `Form.List` 去完成这个功能。

简单说说这个功能，首先是一个动态的表单，可以通过按钮进行增减操作，但是再嵌套了一层，就形成了 `Form.List` 再套 `Form.List` 了。

比如第一个动态表单可以通过外层的按钮去添加同级的表单，嵌套的一层就是在同级的表单里面再套了一 `Form.List`，独属于这个模块下面的动态表单，然后这个下面的动态表单选择的内容又需要根据外层的表单的选中的值进行变化，并且需要同级选择的内容互斥。

这就是一个正常的操作，让我卡壳许久的则是，根据填写的内容，请求后端服务器，用到返回回来的数据进行对这个嵌套表单内的某个选择器或者输入框进行规则校验，表示这个是重复的。好了，下面先简单搭建一下这个动态表单。

## 二、搭建一个嵌套的动态表单

### 2.1 创建一个 `AddForm.tsx` 文件，大概代码如下：

```typescript
const AddForm = memo(() => {
  const [form] = Form.useForm()
  const storeData = Form.useWatch('stores', form)

  const handleSubmit = async () => {
    try {
      const formData = await form.validateFields()
      console.log('formData: ', formData)
    } catch (error) {}
  }

  return (
    <AddFormContext.Provider value={{ form, storeData }}>
      <Form autoComplete='off' form={form} initialValues={initialValues}>
        <Form.List name='stores'>
          {(storesFields, { add: addStore, remove: removeStore }) => (
            <>
              {storesFields.map(storeField => (
                <SelectStore key={storeField.key} storeField={storeField}>
                  {storesFields.length !== 1 && (
                    <Button
                      style={{
                        marginLeft: 16,
                      }}
                      type='primary'
                      onClick={() => removeStore(storeField.name)}
                    >
                      删除商户
                    </Button>
                  )}
                </SelectStore>
              ))}
              <Button
                type='primary'
                icon={<PlusOutlined />}
                onClick={() => addStore(shops)}
                ghost
              >
                添加商户
              </Button>
            </>
          )}
        </Form.List>
      </Form>
      <Button style={{ marginTop: 16 }} type='primary' onClick={handleSubmit}>
        提交表单
      </Button>
    </AddFormContext.Provider>
  )
})

export default AddForm
```

### 2.2 创建第一层动态表单 `components/SlectStore/index.tsx`，代码如下：

```typescript
const SelectStore: FC<SelectStoreProps> = memo((props) => {
  const { storeField, storesFields, remove, children } = props;
  const [storeId, setBizBrandId] = useState<number>(0);

  return (
      <div style={{ padding: 12, border: "1px solid #ccc", marginBottom: 16 }}>
        <Row>
          <Col>
            <Form.Item
              name={[storeField.name, "storeId"]}
              label="商户"
              rules={[{ required: true, message: "请选择" }]}
            >
              <Select
                allowClear
                placeholder="请输入关键词选择"
                style={{ width: 200 }}
              >
                {storeList?.map((item) => (
                  <Select.Option key={item.storeId} value={item.storeId}>
                    {item.storeName}
                  </Select.Option>
                ))}
              </Select>
            </Form.Item>
          </Col>
          <Col>
            <Form.Item>
              {children}
            </Form.Item>
          </Col>
        </Row>
        <Form.List {...storeField} name={[storeField.name, "shops"]}>
            二层嵌套的表单内容。。。。
        </Form.List>
      </div>
  );
});

export default SelectStore;

```

根据以上两个文件，就可以直接实现出一个一层的动态增减表单了，效果如下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d9ebf5692ed416ba93731bbd9d3a235~tplv-k3u1fbpfcp-watermark.image?)

### 2.3 再创建第二层动态表单，代码如下：

```
// components/SlectStore/index.tsx 新增 Form.List 内容
  <Form.List {...storeField} name={[storeField.name, 'shops']}>
    {(shopsFields, { add: addShop, remove: removeShop }) => (
      <div>
        {shopsFields.map(shopField => (
          <SelectShop
            storeId={storeId}
            key={shopField.key}
            shopField={shopField}
          >
            {shopsFields.length !== 1 && (
              <Button
                type='link'
                onClick={() => removeShop(shopField.name)}
              >
                删除门店
              </Button>
            )}
          </SelectShop>
        ))}
        <a onClick={() => addShop()}>
          <PlusOutlined className='mr-[4px]' />
          添加门店
        </a>
      </div>
    )}
  </Form.List>

// components/SlectShop/index.tsx 新增组件
const ShopSelect: FC<ShopSelectProps> = memo(props => {
  const { storeId, shopField, shopFields, remove, children } = props

  const context = useContext(AddFormContext)

  const shopList = useMemo(() => {
    const key = `store${storeId}`
    return storeIdToshop[key]
  }, [storeId])

  return (
    <Row gutter={16}>
      <Col>
        <Form.Item
          name={[shopField.name, 'shopId']}
          label='选择门店'
          rules={[...rules]}
        >
          <Select allowClear placeholder='请输入关键词选择' style={{ width }}>
            {shopList?.map((item: any) => (
              <Select.Option key={item.shopId} value={item.shopId}>
                {item.shopName}
              </Select.Option>
            ))}
          </Select>
        </Form.Item>
      </Col>
      <Col>
        <Form.Item
          name={[shopField.name, 'shopName']}
          label='自定义门店'
          rules={[...rules]}
        >
          <Input allowClear placeholder='请输入' style={{ width }} />
        </Form.Item>
      </Col>
      {children}
    </Row>
  )
})

export default ShopSelect
```

现在就实现了二级嵌套表单的功能了，效果如下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40faa2043bba480bac5db10d99591af5~tplv-k3u1fbpfcp-watermark.image?)

## 三、动态表单搭建完成后，下面实现业务中所需要的一些功能

- 单个模块选择商户对应不同的门店
- 同级商户选择器、门店选择器需要互斥，不能选择一样的
- 根据后端返回的内容，去找对应的某个表单进行提示校验

### 3.1 选择商户对应不同的门店

这里写了一个假数据表示对应不同的门店，根据选择的商户id来区分拥有的门店：

```
export const storeIdToshop = {
  store1: [
    {
      shopId: 100,
      shopName: '门店100',
    },
    {
      shopId: 101,
      shopName: '门店101',
    },
    {
      shopId: 102,
      shopName: '门店102',
    },
    {
      shopId: 103,
      shopName: '门店103',
    },
  ],
  store2: [
    {
      shopId: 200,
      shopName: '门店200',
    },
    {
      shopId: 201,
      shopName: '门店201',
    },
  ],
  store3: [
    {
      shopId: 300,
      shopName: '门店300',
    },
  ],
}
```

在 `SelectStore` 组件中的商户选择器，通过 `onChange` 事件来设置现在选中的商户 `id`，将其存储在一个 `state` 中，再将这个 `state` 传递至子组件 `SelectShop` 当中，子组件根据这个 `id` 去渲染对应的门店列表，代码如下：

```
  const shopList = useMemo(() => {
    const key = `store${storeId}`
    return storeIdToshop[key]
  }, [storeId])
```

根据父组件传递的 `storeId` 属性，去渲染对应的门店列表，如果是需要根据商户 `id` 请求新的门店列表，则实现方法类似。

这样选择不同的商户，所对应的门店也是不同的。效果如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34b83e87294a4b15bc08f7e8509ed4b2~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71b983eb6a7b4f78a5987d7a80d39a3c~tplv-k3u1fbpfcp-watermark.image?)

这里我们还要稍微优化一下，因为目前状态是当商户发生改变的时候，门店选择器如果处于已选择的状态，会保留上次选择商户的门店信息，这时我们应该清空选择器，并且渲染对应的门店类别，让用户去重新选择。代码如下：

```
  const handleBrandChange = (val: number) => {
    setStoreId(val)
    const formKey = ['stores', storeField.name, 'shops']
    // 获取到需要修改的门店列表
    const shopsValue = context?.form?.getFieldValue(formKey)
    const newVal = shopsValue?.map(() => ({
      shopId: undefined,
      shopName: undefined,
    }))
    // 重置
    context?.form.setFieldValue(formKey, newVal)
  }
```

根据 `context` 传递的 `form` 实例，我们可以直接调用 `getFieldValue` 去获取到当前选择商户的这个模块下所有门店列表的信息，然后将其重置为 `undefined`，最后再通过 `setFieldValue` 方法设置回表单。

### 3.2 同级商户、门店选择器进行互斥

商户和门店都需要使用到最外层 `AddForm` 中监听的 `stores` 表单数据，通过 `React.CreateContext` 创建一个 `context` 将 `form`、`storeData` 数据都传递下来，以便于子组件的使用。

可以在商户选择器的 `Option` 中进行设置他的 `disabled` 是否禁用，代码如下：

```typescript
  const disabledStore = useCallback(
    (id: number) => {
      const index = context?.storeData?.findIndex(item => item.storeId === id)
      return index >= 0
    },
    [context?.storeData],
  )
  // 在 Select.Option 中使用
  disabled={disabledStore(item.storeId)}
```

效果如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69d9bf9a349041b3a942d1e44d22a9cd~tplv-k3u1fbpfcp-watermark.image?)

同级门店互斥类似，只是需要多传递一个父级商户的索引值，我们这里为 `storeField.name`
父级传入 `storeField.name` 到 `SelectShop` 组件中，然后直接设置门店选择器的 `Option` 中 `disabled`，代码如下：

```typescript
  const disabledShop = useCallback(
    (id: number) => {
      const index = context?.storeData?.[storeField.name]?.shops?.findIndex(
        item => item?.shopId === id,
      )
      return index >= 0
    },
    [storeField, context?.storeData],
  )
  // 在 Select.Option 中使用
  disabled={disabledShop(item.shopId)}
```

效果如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/485992810a414339b331bc7e06bc3624~tplv-k3u1fbpfcp-watermark.image?)

## 四、重点实现动态校验的功能

这个需求的主要是点击提交表单的时候，会校验是否必填项，但是通过表单校验后就会直接请求后端服务器，系统内部中可能存在选择门店、自定义门店的重复问题，这个时候后端就会给我返回具体的信息，根据这些信息我需要去给某个表单提示校验，响应信息如下：

```
{
    code: 0,
    message: '操作成功',
    data: {
      errorType: 'shop' || 'name',
      storeId: 1,
      shopId: 200,
      failMsg: '【炙·门店2】关联失败，门店名重复',
    },
  }
```

我们根据后端返回的响应信息，自己做一个模拟的异步请求，代码如下：

```
const getRandom = (num: number) => Math.ceil(Math.random() * num);
const request = () =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      const num = getRandom(10)
      const data = {
        // "shop" 表示校验门店选择 || "name" 表示校验自定义门店
        code: num >= 5 ? 200 : 0, // 200表示成功
        errorType: getRandom(10) >= 5 ? 'shop' : 'name',
        storeId: 1,
        shopId: 100,
        failMsg: '【门店100】关联失败，门店名重复',
      }
      resolve(data)
    }, 2000)
  })
```

这里主要使用随机数来判断请求失败与成功，在失败当中再次使用随机数来判断后端返回的是需要校验选择门店还是自定义门店，这里就固定了商户 `id` 以及门店 `id`。

在点击提交表单按钮的试试，首先会进行表单的全部校验，校验必填项，其次发送请求提交表单，我们使用模拟的请求接口，代码如下：

```
const handleSubmit = async () => {
  try {
    const formData = await form.validateFields()
    console.log('formData: ', formData)
    setLoading(true)
    const data: any = await request()
    if (data?.errorType) {
      setRepeatShop(data)
      message.error(data.failMsg)
    } else {
      message.success('提交成功')
      setRepeatShop(null)
    }
    setLoading(false)
  } catch (error) {}
}
```

这里的 `formData` 是传递给后端的具体数据，这里我们使用模拟接口就不用传递数据，接口请求成功后将清除记录的重复门店状态，请求失败则使用 `setRepeatShop` 方法记录门店状态并提示错误信息。

这里 `repeatShop` 重复门店状态会通过 `context` 传递下去。

这里有两种方法实现对某个表单做校验，因为业务需求只用对某个表单进行一个红框提示就好了，所以第一种方法直接通过 `css` 样式来解决这个问题。

### 方法一：通过 `css` 样式实现

在 `SelectShop` 组件中，我们通过 `context` 刚才传递下来的 `repeatShop` 重复门店信息，在 `storeData` 整个表单的数据信息里面去查找对应的索引，具体代码如下：

```
const errorIndex = useCallback(
  (shopIndex: number) => {
    const { storeData, repeatShop } = context || {}
    if (!storeData) return
    // 商户索引
    const index = storeData.findIndex(
      item => item?.storeId === repeatShop?.storeId,
    )
    if (index < 0) return null
    // 门店索引
    const indey = storeData[index]?.shops?.findIndex(
      item => item?.shopId === repeatShop?.shopId,
    )
    if (indey < 0) return null
    console.log(111, storeField.name === index && shopIndex === indey)
    return storeField.name === index && shopIndex === indey
  },
  [context, storeField],
)
```

从 `storeData` 中 根据 `repeatShop` 中 的 `storeId` 捞出商户的索引，然后根据商户索引直接找到对应的 `shops` 门店列表，再根据 `repeatShop` 中的 `shopId` 捞出门店的索引，最后使用商户索引和门店索引与父组件中的 `storeField.name` 当前门店项的 `shopIndex`(传递过来的 `shopField.name`) 进行对比。

返回对比的布尔值，然后通过 `errorIndex` 再去对应的选择门店和自定义门店中的 `Form.Item` 动态添加 `className`，代码如下：

```
// 选择门店
<Form.Item
  ......
  className={`${
    errorIndex(shopField.name) &&
    context?.repeatShop?.errorType === 'shop' &&
    'shop-select-error'
  }`}
>
  ......
</Form.Item>

// 自定义门店
<Form.Item
  ......
  className={`${
    errorIndex(shopField.name) &&
    context?.repeatShop?.errorType === 'name' &&
    'shop-select-error'
  }`}
>
  ......
</Form.Item>
```

通过 `errorIndex(shopField.name)` 调用这个方法返回 `true` 则继续判断重复门店中的 `errorType` 是为 `shop` 还是 `name`，最后动态添加出对应的 `select` 和 `input` 的 `class` 样式类名，具体样式

```
.shop-select-error {
  // select 选择框
  .ant-select-selector {
    border: 1px solid red !important;
  }
}

.shop-input-error  {
  // input 输入框
  .ant-input-affix-wrapper{
    border: 1px solid red;
  }
}
```

最后我们点击提交，接口请求失败后根据返回的信息，动态添加表示错误的红框进行提醒用户，并且会有提示信息。效果如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbcd75ae98f44d609211103919e6e2ae~tplv-k3u1fbpfcp-watermark.image?)

### 方法二：通过组件自带的 `form.validateFields()` 方法实现

在提交表单的方法中，我们需要再添加一个全局校验方法 `form.validateFields()`，代码如下：

```
const handleSubmit = async () => {
  try {
    const formData = await form.validateFields()
    console.log('formData: ', formData)
    setLoading(true)
    const data: any = await request()
    if (data?.errorType) {
      setRepeatShop(data)
      message.error(data.failMsg)
      await form.validateFields()
    } else {
      message.success('提交成功')
      setRepeatShop(null)
    }
    setLoading(false)
  } catch (error) {}
}
```

下面我们修改 `SelectShop` 组件当中的代码，首先我们需要自定义一个校验方法，代码如下：

```
const customValidator = (id: number, type: "shop" | "name") => {
  const { shopId, errorType } = context?.repeatShop || {};
  if (shopId === id && errorType === type) {
    return Promise.reject(new Error("门店名重复"));
  }
  return Promise.resolve();
};
```

根据这个方法，去给选择门店和自定义门店的 `Form.Item` 中添加 `rules`，代码如下：

```
// 选择门店
rules={[
    ...rules,
    {
      validator(rule, value) {
        return customValidator(value, DailyMappingError.SelectError)
      },
    },
  ]}
// 自定义门店
rules={[
    ...rules,
    {
      validator() {
        const fieldKey = [
          'stores',
          storeField.name,
          'shops',
          shopField.name,
          'shopId',
        ]
        const id = context?.form?.getFieldValue(fieldKey)
        return customValidator(id, 'name')
      },
    },
  ]}
```

最后我们再次提交表单，当点击提交表单后，后端返回新的信息后，我们再次进行了一次校验 `form.validateFields()`，效果如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a1151a604d64fa191fe9f2f31e042e8~tplv-k3u1fbpfcp-watermark.image?)

## 五、结尾

案例代码：
[Antd Form.List 动态嵌套多级表单及根据内容重新校验某个表单 - CodeSandbox](https://codesandbox.io/s/antd-form-list-dong-tai-qian-tao-duo-ji-biao-dan-ji-gen-ju-nei-rong-chong-xin-xiao-yan-mou-ge-biao-dan-hseizt?file=/src/AddForm.tsx)

以上内容有问题的，望各位大佬指点，谢谢~
