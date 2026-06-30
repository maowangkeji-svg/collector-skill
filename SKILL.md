---
name: marketplace-collector
description: 用于构建、修改和排查跨平台电商商品采集器，适用于 Shopee、Temu、MercadoLibre、MercadoLivre、Amazon、AliExpress、eBay、Lazada 等平台。Use when Codex needs to add marketplace platform adapters, product list/detail parsing, queue and concurrency behavior, collection logs, partial-success persistence, Excel/JSON export, public contact extraction, image handling, pagination, duplicate checks, or collector UI for browser extensions and scraping/collection scripts.
---

# 跨平台商品采集器

使用本 skill 开发或适配电商平台商品采集器。优先遵循当前项目已有结构、代码风格和工具链，不要一上来重写框架。

## 核心流程

1. 先熟悉仓库，不假设文件名固定。

```powershell
rg -n "manifest|content_scripts|chrome\\.runtime|queue|export|xlsx|json|collector|collect|product|Shopee|Temu|Mercado|Amazon|AliExpress"
rg --files
```

2. 判断当前采集器形态：

- 浏览器扩展 content script
- background / service worker 请求桥
- 页面注入脚本
- 后端保存 / 查重接口
- 纯本地导出工具

3. 把平台差异逻辑和通用采集逻辑拆开。可以的话，每个平台维护一个 adapter。

4. 坚持“能采多少保存多少”。详情页失败时，也要保存列表卡片里已有的标题、价格、图片、链接、商品 ID 和错误信息。

5. 队列执行要可控。默认并发保守，用户配置的并发数必须做上下限约束；日志要限制条数，避免长期运行导致内存增长。

6. 修改 JavaScript 后至少做语法检查：

```powershell
node --check <content-script>
node --check <background-script>
```

## 安全与边界

- 只采集公开页面数据，或当前浏览器会话中用户可见的数据。
- 不绕过登录、验证码、风控、频率限制、付费墙或账号权限限制。
- 不采集隐藏的个人隐私数据。邮箱、手机号等联系方式只能从页面公开可见文本或商品描述中提取。
- 不硬编码私有 host、租户域名、Cookie、token 或本地路径。
- 不默认高并发请求。默认并发要低，只有用户明确需要时才开放设置。

## 参考文件

按任务需要读取，不要一次性加载全部：

- `references/platform-adapter.md`：新增或排查 Shopee、Temu、MercadoLibre、Amazon、AliExpress、Lazada 等平台适配器时读取。
- `references/fields-schema.md`：修改采集字段、统一商品数据结构、Excel 列或 JSON 结构时读取。
- `references/queue-export.md`：修改队列、并发、日志、部分成功保存、持久化、Excel 或 JSON 导出时读取。

## 实现原则

- 优先读结构化数据：平台初始状态、JSON-LD、内嵌 app data、meta 标签。
- DOM 选择器作为兜底，选择器要兼顾稳定性，不要只依赖一个实验样式类名。
- 搜索页 / 列表页采集时，优先保存卡片上的标题、链接、价格、币种、主图、商品 ID。
- 详情页采集时，把结构化状态、JSON-LD、DOM、meta、列表卡片兜底数据合并。
- 图片要做标准化，兼容 `srcset`、`data-src`、`data-srcset`、`data-original`、平台图片 ID 等懒加载形式。
- 查重要按平台、商品 ID、标准化 URL 组合判断。
- 导出要稳定：Excel 适合扁平字段和人工查看；JSON 保留数组和对象，方便再次导入。

## 常见用户需求

- “给这个扩展增加 Shopee 采集。”
- “让采集器支持 Temu。”
- “除了 Excel，也支持导出 JSON。”
- “详情页失败时，也要保存能采到的基础信息。”
- “增加采集日志和并发采集。”
- “采集商家名称、商家 ID、销量、库存、上架时间、属性、变体、公开联系方式。”
