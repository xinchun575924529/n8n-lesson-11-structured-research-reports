# 02 · 坑集（全部实锤，含验证方法）

## 🔴 结构级（B 关 mock 实锤）

### 坑1 · 报告按词条逐条生成，不是聚合一次
循环回连接在末端节点上，尾巴整条都在循环体内。N 条结果 = N 次 AI 综合 + N 条 Slack + N 次写表。
**后果**：API 费用 ×N、Slack 刷屏、调用方只能收到第一条响应。
**加固**：把回连点改到 `Clean Website Content`，尾巴整体挪出循环（见 docs/05）。

### 坑2 · 只处理第一个查询
`Prepare Search Results` 用 `$input.first()`，5 个查询只跑第 1 个。
**加固**：改 `$input.all()` 并对每项取 `item.json.query.search` 展平。

### 坑3 · Sheets 的 Topic 列存的是词条标题
`Research Report Formatter` 从 `$('Merge Research Data').first().json.title` 取 topic——那是**词条标题**不是研究主题。同主题跑两次，表里是两行不同词条。
**加固**：从 `$('Receive Research Topic').first().json.topic` 取。

## 🟡 单元级（A 关 30 断言实锤）

### 坑4 · Parse Search Queries 裸 JSON.parse 无 try
LLM 加一句开场白、返回空、返回单引号伪 JSON——三种常见脏输出**全部直接崩执行**。
**加固**：try/catch + 正则 `\[[\s\S]*\]` 提取数组。

### 坑5 · 报告切片完全依赖 `###` 前缀
LLM 改用 `##` 或加粗标题，8 个字段**全部静默变空串**，不报错，Sheets 里存一堆空列。
**C 关真跑复现**：DeepSeek 默认就不输出 `###`——模板在 GPT-4o-mini 上能跑，纯粹是吃了那个模型的隐性习惯。
**加固**：prompt 里锁死「每个章节标题必须用 ### 前缀」（实测一行就修好）。

### 坑6 · Wikipedia 查不到时静默产空内容
`page?.extract || ''`——查不到不报错，空内容直接进 AI 综合，稀释报告质量。
**加固**：Clean 后加 If 过滤空 content。

### 坑7 · 空 topic 静默断链
Validate 的 false 分支没接任何节点——调用方干等到超时，无任何错误提示。
**加固**：false 分支接 respondToWebhook 返回 400。

### 坑8 · 词条正文不截断直接灌 prompt
Wikipedia 长文几万字，直接进 LLM 上下文。
**加固**：`.slice(0, 12000)` 截断守护。

## 🟠 环境级（C 关真跑实锤）

### 坑9 · Wikipedia 国内不可达
本机直连 000、WARP 也救不了（DNS endpoint 路由异常）。百度百科/豆瓣系服务端抓取全触发反爬挑战。
**出路**：DBpedia lookup（可达免费无 key，但公共端点数据集被裁剪、无 abstract）或海外节点。

### 坑10 · n8n httpRequest jsonBody 表达式陷阱
`={"model":...}` 对象字面量、`=JSON.stringify(...)` 都被 parseJsonParameter 拒（"not valid JSON"）。
**唯一稳的姿势**：上游 Code 节点预构 JSON 字符串，jsonBody 写 `={{ $json.reqBody }}`。

## 灰榜（观察项，不算 bug）

- AI 输出为空时抛 `AI research report is empty.`——难得一见的友好报错
- 章节乱序时切片不会吞内容（nextHeadings 是后续章节并集）——比看起来健壮
- 节点名 `z`——大型工作流里的命名灾难