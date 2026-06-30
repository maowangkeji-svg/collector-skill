# Marketplace Collector Skill 使用教程

这个 skill 已创建在：

```text
C:\Users\Tiger\.codex\skills\marketplace-collector
```

## 什么是 Skill

Skill 是给 Codex/AI 使用的一组“专业工作说明”和“参考资料”。它会告诉 Codex：遇到某类任务时应该怎么分析、要读哪些参考、按什么流程实现、要注意哪些边界。

可以把 skill 理解成：

- 一份给 AI 的开发手册
- 一套可复用的任务流程
- 一组领域经验和检查清单
- 一个让 Codex 更懂某类任务的能力包

## Skill 不是什么

这个 skill **不是一个完整的采集插件**，也不是安装后就能直接采集 Shopee、Temu、Amazon 的浏览器扩展。

它不包含：

- 可直接安装到 Chrome 的完整扩展包
- 针对所有平台已经写好的采集代码
- 自动绕过平台风控、登录、验证码的能力
- 内置账号、Cookie、token 或私有接口权限

换句话说，skill 本身不会替你采集商品。它的作用是：当你让 Codex 开发或修改采集器时，Codex 会按照这套说明更快、更规范地写代码。

## Skill 和采集插件的区别

| 对象 | 作用 | 谁使用 |
| --- | --- | --- |
| Skill | 指导 Codex 如何开发采集器 | Codex / AI |
| 浏览器插件 | 真正运行在 Chrome 里采集商品 | 用户 |
| 采集脚本 | 执行具体采集逻辑 | Node、浏览器或后端 |
| 后端接口 | 保存、查重、限流、导出等服务 | 采集器调用 |

举例：

```text
marketplace-collector skill
```

不是 Shopee 采集插件。

但你可以对 Codex 说：

```text
使用 marketplace-collector skill，帮我把当前浏览器扩展改造成支持 Shopee 采集。
```

然后 Codex 会根据这个 skill 里的流程和规范，去阅读项目、设计 adapter、修改代码、验证功能。

## 它能做什么

这个 skill 是给 Codex/AI 用的通用采集开发指南。别人拿到后，可以让 Codex 按同一套方法开发或修改 Shopee、Temu、MercadoLibre、Amazon、AliExpress、Lazada 等平台的商品采集插件。

它不是一个直接安装就能采集所有平台的浏览器插件，而是帮助 Codex 更快、更规范地写出对应平台适配代码。

它现在包含更详细的采集原理、平台 adapter 设计、详情 API 适配、商城地址配置、字段映射、队列并发、日志和导出规范。

## 如何分享给别人

把整个文件夹发给对方：

```text
C:\Users\Tiger\.codex\skills\marketplace-collector
```

对方放到自己的 Codex skills 目录，例如：

```text
C:\Users\对方用户名\.codex\skills\marketplace-collector
```

然后重启 Codex 或开启新线程即可自动发现。

![Skill 导入流程](docs/images/skill-install-flow.svg)

## 如何触发使用

触发 skill 最稳的方法是在消息里直接点名 skill 名称。

这个 skill 的名称是：

```text
marketplace-collector
```

### 方式一：使用普通名称触发

可以直接在请求里写：

```text
使用 marketplace-collector skill，帮我给这个浏览器插件增加 Shopee 商品采集能力。
```

### 方式二：使用 `$skill-name` 触发

也可以用 `$` 形式写：

```text
使用 $marketplace-collector，帮我把当前采集器改成支持 Temu。
```

### 方式三：靠任务描述自动触发

如果你的描述足够匹配，Codex 也可能自动触发：

```text
帮我给这个浏览器插件增加 Shopee 商品采集，支持队列、并发、日志、Excel 和 JSON 导出。
```

不过为了稳定，推荐明确写上：

```text
使用 marketplace-collector skill
```

![Skill 触发提示词](docs/images/skill-trigger-prompt.svg)

## 如何确认已经触发

通常 Codex 会在开始工作时说类似：

```text
我会使用 marketplace-collector skill 来处理这个跨平台采集任务。
```

或者它会先读取：

```text
C:\Users\你的用户名\.codex\skills\marketplace-collector\SKILL.md
```

如果 Codex 没有提到 skill，也没有读取 `SKILL.md`，可以直接补一句：

```text
请明确使用 marketplace-collector skill 重新处理。
```

## 触发失败怎么办

如果没有触发，按下面检查：

1. 确认目录存在：

```text
C:\Users\你的用户名\.codex\skills\marketplace-collector\SKILL.md
```

2. 确认 `SKILL.md` 里 frontmatter 有：

```yaml
---
name: marketplace-collector
description: ...
---
```

3. 重启 Codex，或新开一个线程。

4. 在提示词里明确写：

```text
使用 marketplace-collector skill
```

## 常用提示词

### 信息不足时先让 Codex 提问

```text
使用 marketplace-collector skill，我想支持一个新的电商平台采集。
请先告诉我需要提供哪些信息，不要先写代码。
```

### 按模板提供平台信息

```text
使用 marketplace-collector skill，帮我适配一个新平台。

平台：Shopee
国家/地区：新加坡
商城首页：https://shopee.sg
搜索页示例：https://shopee.sg/search?keyword=phone
商品详情页示例：https://shopee.sg/xxx-i.123456.789012
商品详情 API：暂时没有
是否需要登录：不需要
需要字段：
- 标题
- 价格
- 币种
- 主图
- 全部图片
- 描述
- 销量
- 库存
- 上架时间
- 商家名称
- 商家ID
- 属性
- 变体
导出格式：Excel、JSON
并发要求：默认 1，可设置 1-5
```

### 提供商品详情 API

```text
使用 marketplace-collector skill，帮我把商品详情解析改成优先使用详情 API。

商城首页：https://example-market.com
商品详情页示例：https://example-market.com/product/123
详情 API 示例：https://example-market.com/api/product/detail?id=123
请求方法：GET
是否需要登录：不需要
响应 JSON 示例：
{
  "item": {
    "id": "123",
    "title": "Demo product",
    "price": "19.99",
    "currency": "USD",
    "images": ["https://example.com/a.jpg"],
    "stock": 20,
    "sold_count": 88,
    "seller": { "id": "s001", "name": "Demo Shop" }
  }
}

请建立字段映射，API 失败时回退页面 DOM，并同步 Excel/JSON 导出。
```

注意：不要把真实 Cookie、Authorization、token、CSRF、账号 ID、设备指纹发给 Codex。如果必须说明鉴权方式，请打码。

### 添加 Shopee

```text
使用 marketplace-collector skill，熟悉当前项目，然后添加 Shopee 商品采集。
要求支持搜索页商品入队、详情解析、图片、价格、销量、库存、店铺名称、店铺ID，并支持 Excel/JSON 导出。
```

### 添加 Shopee 并保留现有逻辑

```text
使用 marketplace-collector skill，在不破坏现有 MercadoLibre 采集能力的前提下，新增 Shopee 平台 adapter。
要求共用队列、日志、Excel/JSON 导出，但平台解析逻辑要独立。
```

### 添加 Temu

```text
使用 marketplace-collector skill，给当前浏览器扩展增加 Temu 商品采集适配器。
要求能采多少保存多少，详情页失败也要保存列表页基础信息。
```

### 添加 Amazon

```text
使用 marketplace-collector skill，给当前采集器增加 Amazon 商品详情页采集。
先支持详情页标题、价格、币种、主图、店铺名称、库存状态、描述和 JSON 导出。
不要采集账号私有信息。
```

### 增加字段

```text
使用 marketplace-collector skill，给采集结果增加品牌、评分、评论数、店铺评分字段，并同步导出 Excel 和 JSON。
```

### 增加公开邮箱提取

```text
使用 marketplace-collector skill，增加公开邮箱提取字段。
只允许从页面可见文本、商品描述、商家公开介绍中提取邮箱，不要绕过平台限制。
Excel 和 JSON 都要导出 emails 字段。
```

### 优化队列

```text
使用 marketplace-collector skill，把采集队列改成支持可配置并发、固定高度日志、最多保留100条日志。
```

### 修复图片

```text
使用 marketplace-collector skill，排查采集结果图片为空的问题。
重点检查懒加载图片、srcset、data-src、平台图片ID和导出前图片URL标准化。
```

### 部分成功保存

```text
使用 marketplace-collector skill，修改采集逻辑：详情页请求失败时也要保存搜索页卡片已有数据，并把错误信息写入 errorMessage。
```

## 适配新平台的推荐流程

1. 先让 Codex 熟悉项目结构。
2. 确认是浏览器扩展、脚本工具，还是后端采集器。
3. 提供商城首页、搜索页示例、商品详情页示例。
4. 如果有商品详情 API，提供 API 地址、请求方法、参数来源、响应示例。
5. 新增平台 adapter。
6. 先采列表页基础字段。
7. 再采详情页字段或详情 API 字段。
8. 失败时保存部分成功记录。
9. 最后同步 Excel/JSON 导出。

## 建议你提供的信息

```text
平台：
国家/地区：
商城首页：
搜索页示例：
商品详情页示例：
商品详情 API（如果有）：
请求方法（如果有 API）：
API 参数如何从商品 URL 生成：
响应 JSON 示例（敏感信息打码）：
是否需要登录：
需要字段：
导出格式：
并发要求：
备注：
```

如果你只提供“支持 Shopee / 支持 Temu”，Codex 应该先问你要这些信息，而不是直接编造接口地址。

## 注意事项

- 只采集页面公开展示或当前登录用户可见的数据。
- 不要绕过登录、验证码、风控、付费墙或平台限制。
- 邮箱、手机号等联系方式只能从公开可见文本中提取。
- 并发默认要保守，避免请求过快导致页面异常或账号风险。
