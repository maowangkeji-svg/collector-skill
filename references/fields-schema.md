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
- 如果字段来自商品详情 API，保留必要的 `raw` 片段或字段映射注释，方便后续接口变化时排查。
- Excel 导出使用扁平列：
  - `seller.name` -> `sellerName`
  - `seller.id` -> `sellerId`
  - `images` -> 换行分隔 URL
  - `attributes` -> 换行分隔 `name:value`
  - `variants` -> 换行分隔变体名，或紧凑 JSON
- 部分字段解析失败时，保留 `errorMessage`，不要丢弃整条商品。

## 详情 API 字段映射

当用户提供 API 响应 JSON 时，先建立字段映射表，再写解析代码。

示例：

```text
API 字段                  统一字段
item.item_id              productId
item.title                title
item.price                price
item.currency             currency
item.images[]             images
item.stock                stock
item.sold_count           sales
item.shop.shop_id         seller.id
item.shop.name            seller.name
item.attributes[]         attributes
item.skus[]               variants
```

映射原则：

- 价格可能是整数分、字符串或带小数的数字，要确认单位。
- 图片可能是完整 URL、相对路径或平台图片 ID，要标准化。
- 库存可能在商品级，也可能在 SKU / variant 级。
- 销量可能是区间、文本或真实数字，按页面展示口径保存。
- 上架时间可能是时间戳、ISO 字符串或本地化日期文本。
- 商家 ID 可能叫 shopId、sellerId、merchantId、storeId。

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
