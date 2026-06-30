# 队列、日志与导出参考

修改队列执行、并发、日志、持久化或导出时读取本文件。

## 队列模型

推荐状态：

```javascript
const queue = [];
const activeTasks = [];
const records = [];
const logs = [];
const MAX_LOG_COUNT = 100;
let processingPromise = null;
```

每个任务要包含列表卡片兜底数据：

```javascript
{
  id: "",
  platform: "",
  url: "",
  productId: "",
  title: "",
  image: "",
  price: "",
  currency: "",
  status: "waiting",
  message: "",
  createdAt: ""
}
```

如果需要请求商品详情 API，任务中还可以保存从列表页解析出的接口参数：

```javascript
{
  productId: "",
  sellerId: "",
  shopId: "",
  skuId: "",
  region: "",
  apiParams: {}
}
```

这些字段用于生成详情 API 请求，但不要保存 Cookie、token、签名密钥等敏感信息。

## 并发

- 默认并发为 `1`。
- 只有用户要求，或项目已经有设置入口时，才暴露并发配置。
- 并发数要做上下限约束，例如 `1..10`。
- 使用 `activeTasks` 单独保存正在采集的任务，方便 UI 展示。
- 重复点击开始时，复用同一个 `processingPromise`，不要启动多套 worker。

推荐模式：

```javascript
async function processQueue() {
  if (processingPromise) return processingPromise;
  processingPromise = runQueue();
  try {
    await processingPromise;
  } finally {
    processingPromise = null;
  }
}
```

## 部分成功

详情页解析失败时：

1. 用 task 中的列表卡片数据生成部分成功记录。
2. 保存错误信息。
3. 继续处理队列。

单个字段解析失败时：

1. 捕获字段错误。
2. 该字段留空。
3. 保存其他字段。

## 日志

除非用户明确要求持久化，否则日志只放内存。

建议记录：

- 加入队列
- 开始采集
- 采集成功
- 采集失败 / 部分成功
- 进度：已采集、剩余、队列、正在采集
- 翻页、自动采集完成、无下一页

日志数量上限：

```javascript
function addLog(type, message) {
  logs.push({ type, message, time: formatTime(new Date()) });
  if (logs.length > MAX_LOG_COUNT) {
    logs.splice(0, logs.length - MAX_LOG_COUNT);
  }
}
```

UI 建议：

- 固定高度日志框。
- 黑色背景，方便快速扫描。
- 如果用户要求，日志可按最新在上排序。
- 不要展示 `100/100` 这类内部计数，除非用户明确要求。

## 导出

Excel：

- 使用扁平、人类可读列。
- HTML `.xls` 导出时加 BOM 和 UTF-8 meta。

JSON：

- 保留嵌套对象和数组。
- 包含 `exportedAt`、`total`、`records`。
- 导出前标准化图片 URL。

## 验证

JavaScript 修改后运行：

```powershell
node --check <content-script>
node --check <background-script>
```
