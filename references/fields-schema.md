# 商品字段结构参考

修改采集字段、导出字段或统一商品数据结构时读取本文件。

## 统一商品结构

```javascript
{
  collectedAt: "YYYY-MM-DD HH:mm:ss",
  status: "success | partial_success | failed",
  platform: "shopee | temu | mercadolibre | amazon | ...",
  region: "",
  productId: "",
  parentProductId: "",
  title: "",
  price: "",
  originalPrice: "",
  currency: "",
  currencySymbol: "",
  url: "",
  canonicalUrl: "",
  mainImage: "",
  images: [],
  description: "",
  sales: "",
  stock: "",
  listedAt: "",
  seller: {
    id: "",
    name: "",
    url: "",
    rating: "",
    location: ""
  },
  categories: {
    rootCategoryId: "",
    categoryId: "",
    path: []
  },
  dimensions: {
    length: "",
    width: "",
    height: "",
    weight: ""
  },
  attributes: [
    { "name": "", "value": "" }
  ],
  variants: [
    {
      "id": "",
      "name": "",
      "price": "",
      "stock": "",
      "image": "",
      "attributes": []
    }
  ],
  publicContacts: {
    emails: [],
    phones: [],
    sourceText: ""
  },
  raw: {},
  errorMessage: ""
}
```

## 字段规则

- 未知值用空字符串或空数组，不要写 `undefined`。
- JSON 导出中保留有价值的数组和对象，方便二次导入。
- Excel 导出使用扁平列：
  - `seller.name` -> `sellerName`
  - `seller.id` -> `sellerId`
  - `images` -> 换行分隔 URL
  - `attributes` -> 换行分隔 `name:value`
  - `variants` -> 换行分隔变体名，或紧凑 JSON
- 部分字段解析失败时，保留 `errorMessage`，不要丢弃整条商品。

## 公开联系方式提取

只提取公开可见页面文本或商品描述中出现的联系方式。

邮箱基础正则：

```javascript
/[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi
```

过滤明显误报，例如图片文件名、CSS 资源、平台官方客服模板等非卖家提供的联系方式。

## 常见平台字段

- 销量：sold quantity、订单数量、明确写着 sold / vendido / vendas 等文本。
- 库存：可见库存、SKU 库存、变体库存。
- 上架时间：创建时间、发布时间、listing start time。只有公开时才采。
- 商家 ID：store ID、shop ID、seller ID、merchant ID、店铺 URL 中的 ID。
- 商家名称：店铺名、商家名、profile 显示名。
