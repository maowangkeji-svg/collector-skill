# Marketplace Collector Skill 使用教程

这个 skill 已创建在：

```text
C:\Users\Tiger\.codex\skills\marketplace-collector
```

## 它能做什么

这个 skill 是给 Codex/AI 用的通用采集开发指南。别人拿到后，可以让 Codex 按同一套方法开发或修改 Shopee、Temu、MercadoLibre、Amazon、AliExpress、Lazada 等平台的商品采集插件。

它不是一个直接安装就能采集所有平台的浏览器插件，而是帮助 Codex 更快、更规范地写出对应平台适配代码。

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
3. 新增平台 adapter。
4. 先采列表页基础字段。
5. 再采详情页字段。
6. 失败时保存部分成功记录。
7. 最后同步 Excel/JSON 导出。

## 注意事项

- 只采集页面公开展示或当前登录用户可见的数据。
- 不要绕过登录、验证码、风控、付费墙或平台限制。
- 邮箱、手机号等联系方式只能从公开可见文本中提取。
- 并发默认要保守，避免请求过快导致页面异常或账号风险。
