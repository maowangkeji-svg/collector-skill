# Marketplace Collector Skill

这是一个给 Codex/AI 使用的跨平台电商商品采集开发 skill。

它用于指导 Codex 开发、修改和排查 Shopee、Temu、MercadoLibre、Amazon、AliExpress、Lazada 等平台的商品采集器。

## 重要说明

这个仓库不是一个完整的浏览器采集插件。

它不会在导入后直接采集商品，也不包含已经写好的 Shopee、Temu、Amazon 全平台采集代码。

它的作用是给 Codex 提供一套专业工作说明和参考资料，让 Codex 在你提出采集开发需求时，按统一流程完成：

- 平台页面识别
- 搜索页商品入队
- 商品详情页解析
- 商品详情 API 适配
- 字段归一化
- 部分成功保存
- 队列并发
- 采集日志
- Excel / JSON 导出
- 图片 URL 标准化
- 翻页和去重

## 目录结构

```text
collector-skill/
├── SKILL.md
├── Skill使用教程.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── collection-principles.md
    ├── fields-schema.md
    ├── platform-adapter.md
    ├── queue-export.md
    └── required-info.md
```

## 文件说明

- `SKILL.md`：skill 主入口，包含触发描述、核心流程、安全边界和参考文件导航。
- `Skill使用教程.md`：中文使用教程，包含触发方式、案例提示词、需要提供的信息模板。
- `agents/openai.yaml`：Codex App 中展示 skill 的名称、简介和默认提示词。
- `references/collection-principles.md`：采集原理说明，包括页面解析、接口解析、搜索页、详情页、自动采集和日志。
- `references/required-info.md`：新增平台采集前需要用户提供的信息清单。
- `references/platform-adapter.md`：平台 adapter 设计、商城地址配置、详情 API 适配说明。
- `references/fields-schema.md`：统一商品字段、详情 API 字段映射、公开联系方式提取规则。
- `references/queue-export.md`：队列、并发、日志、部分成功保存和 Excel/JSON 导出规则。

## 如何导入 Codex

把整个 `marketplace-collector` skill 文件夹放到 Codex skills 目录。

Windows 示例：

```text
C:\Users\你的用户名\.codex\skills\marketplace-collector
```

本机示例：

```text
C:\Users\Tiger\.codex\skills\marketplace-collector
```

放好后，重启 Codex 或新开一个线程。

## 如何触发

最稳的方式是在提示词里明确写 skill 名称：

```text
使用 marketplace-collector skill，帮我给当前浏览器扩展增加 Shopee 商品采集能力。
```

也可以使用 `$` 形式：

```text
使用 $marketplace-collector，帮我把当前采集器改成支持 Temu。
```

如果描述足够匹配，Codex 也可能自动触发，但建议显式写出 skill 名称。

## 新平台适配时建议提供的信息

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

如果只说“支持 Shopee”或“支持 Temu”，Codex 应该先向你索要这些信息，而不是编造接口地址或商城地址。

## 示例提示词

### 先询问信息

```text
使用 marketplace-collector skill，我想支持一个新的电商平台采集。
请先告诉我需要提供哪些信息，不要先写代码。
```

### 添加 Shopee

```text
使用 marketplace-collector skill，熟悉当前项目，然后添加 Shopee 商品采集。
要求支持搜索页商品入队、详情解析、图片、价格、销量、库存、店铺名称、店铺ID，并支持 Excel/JSON 导出。
```

### 使用详情 API

```text
使用 marketplace-collector skill，帮我把商品详情解析改成优先使用详情 API。
我会提供商城首页、商品详情页示例、详情 API 地址、请求方法和响应 JSON 示例。
API 失败时需要回退页面 DOM，并同步 Excel/JSON 导出。
```

## 安全边界

- 只采集公开页面数据，或当前浏览器会话中用户可见的数据。
- 不绕过登录、验证码、风控、频率限制、付费墙或账号权限限制。
- 不索要或保存真实 Cookie、Authorization、token、CSRF、设备指纹、账号密码等敏感信息。
- 邮箱、手机号等联系方式只能从公开可见文本中提取。
- 并发默认应保守，避免请求过快导致页面异常或账号风险。

## 详细教程

更多案例和使用方式请看：

```text
Skill使用教程.md
```
