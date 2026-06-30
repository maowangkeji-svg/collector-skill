# 平台适配器参考

新增或排查电商平台采集能力时读取本文件。

## 推荐 Adapter 结构

每个平台尽量维护一个小型 adapter：

```javascript
const adapter = {
  id: "shopee",
  name: "Shopee",
  hosts: ["shopee.com", "shopee.sg", "shopee.com.my"],
  mallUrls: ["https://shopee.sg"],
  detailApi: {
    urlPattern: "",
    method: "GET",
    buildRequest(productUrl, task) {},
    parseResponse(json, task) {}
  },
  isSearchPage(url, doc) {},
  isProductPage(url, doc) {},
  getProductLinks(doc) {},
  parseSearchCard(anchor, doc) {},
  parseProductDetail(doc, url, task, state) {},
  findNextPage(doc, url) {}
};
```

通用逻辑只调用 adapter 方法，再把返回结果归一化成统一商品结构。

如果项目暂时没有 adapter 框架，也要在实现中保持同样边界：平台识别、列表解析、详情解析、翻页、字段归一化不要和通用队列/导出逻辑混在一起。

## 识别顺序

1. 根据 manifest、当前 URL 或项目配置识别平台 host。
2. 根据 URL 规则和商品卡片 DOM 判断搜索页 / 列表页。
3. 根据 URL 规则、商品 ID、JSON-LD Product 对象和详情页 DOM 判断商品详情页。
4. 先解析列表卡片基础数据。
5. 把基础数据放进任务后，再请求或解析详情页。

## 新平台适配步骤

1. 记录平台信息：
   - 平台名
   - 国家 / 地区
   - 商城首页
   - 搜索页 / 分类页示例
   - 商品详情页示例
2. 分析 URL：
   - 搜索页 URL 是否有关键词参数
   - 分类页 URL 是否有分页参数
   - 商品详情页 URL 中是否包含商品 ID、店铺 ID
3. 分析详情数据来源：
   - 页面 HTML 是否有 JSON-LD
   - 是否有 `__NEXT_DATA__`、`__NUXT__`、Redux、Apollo 或平台状态对象
   - 是否有公开商品详情 API
   - DOM 中哪些节点显示目标字段
4. 先实现列表页入队。
5. 再实现详情页字段补齐。
6. 最后接入导出、日志、并发、自动翻页和部分成功保存。

## 商品详情 API 适配

当用户提供商品详情 API 地址时，adapter 可以包含 `detailApi` 配置，但不要硬编码敏感鉴权。

推荐处理：

```javascript
async function fetchDetailByApi(task) {
  const request = adapter.detailApi.buildRequest(task.url, task);
  const response = await fetch(request.url, {
    method: request.method || "GET",
    credentials: "include",
    headers: request.headers || {}
  });
  const json = await response.json();
  return adapter.detailApi.parseResponse(json, task);
}
```

注意：

- `credentials: "include"` 只能用于当前用户正常可访问的数据。
- 不把 Cookie、Authorization、token、签名密钥写进代码。
- API 失败时回退到页面结构化数据和 DOM。
- API 字段要归一化到统一商品结构，不要直接把接口原始字段当最终导出字段。

## 商城地址配置

同一平台不同地区站点要分开配置：

```javascript
const SHOPEE_SITES = [
  { region: "sg", host: "shopee.sg", mallUrl: "https://shopee.sg", currency: "SGD" },
  { region: "my", host: "shopee.com.my", mallUrl: "https://shopee.com.my", currency: "MYR" }
];
```

配置用途：

- 判断页面是否属于目标平台。
- 补充默认币种和地区。
- 生成相对 URL 的绝对地址。
- 判断接口是否同源或需要 host permissions。

## 结构化数据来源

优先读取这些来源，再使用脆弱 DOM 选择器：

- `application/ld+json`
- Next.js `__NEXT_DATA__`
- Nuxt `__NUXT__`
- Apollo / Redux 状态数据
- 平台 app 初始状态变量
- OpenGraph / Twitter meta 标签

搜索内嵌脚本时尽量使用明确 marker，不要盲目解析所有 script。

## DOM 兜底

DOM 解析时，每个字段准备多个候选选择器，取第一个干净值。

常见兜底类别：

- 标题：`h1`、商品标题类名、`og:title`
- 价格：金额组件、促销价节点、JSON-LD 的 `priceCurrency`
- 图片：图库图片、`srcset`、懒加载属性、`og:image`
- 商家：商家面板链接、店铺卡片、店铺主页链接
- 库存 / 销量：包含 sold、available、stock、inventory 等含义的可见文本
- 描述：商品描述区和 meta description

## 平台提示

### Shopee

- 很多页面是 SPA 渲染，且不同国家站点差异较大。
- 商品 URL 常见形式包含 `-i.<shopid>.<itemid>` 或 query ID。
- 优先读取页面可见状态和公开嵌入 JSON。不要绕过反爬或私有 API。
- 店铺名称和 ID 通常可以从 URL、状态数据或可见店铺信息中获取。

### Temu

- 商品页可能严重依赖客户端渲染。
- 优先 JSON-LD、页面状态、meta 标签和可见 DOM。
- 不要假设库存、销量、商家 ID 在每个商品上都会公开。

### MercadoLibre / MercadoLivre

- Nordic initial state 经常包含较完整商品数据。
- 商品 ID 常见前缀：`MLA`、`MLB`、`MLM` 等。
- 图片 ID 可能需要标准化为 `https://http2.mlstatic.com/...`。

### Amazon

- DOM 会随国家站点、实验和类目变化。
- 选择器要保守，避免读取账号相关私有数据。
- 商家和库存字段可能缺失，也可能用不同文案表达。

## 翻页

优先使用平台分页控件。只有在 URL 翻页规则稳定时，才用 offset / page 参数兜底。

自动采集时，不要在当前队列批次完成前翻页；只有目标数量不足时再进入下一页。
