# n8n 实战课 11：AI 研究报告生成器（Structured Research Reports）

> 官方模板：[Generate structured research reports with OpenAI, Wikipedia, Google Sheets, and Slack](https://n8n.io/workflows/19710/)（模板 ID 19710，12 功能节点 + 17 张教学便签）

输入一个研究主题，自动完成「AI 生成搜索查询 → 逐条检索百科 → 抓取正文 → AI 综合 → 8 章节结构化报告 → 存档 + 群通知 + 回传调用方」全流程。

## 你会学到

1. **Webhook 双向通信**：POST 进来、处理完把结构化报告原路返回（responseNode 模式）
2. **两阶段 LLM 用法**：第一阶段把主题拆成 5 个搜索查询，第二阶段把资料综合成报告
3. **splitInBatches 循环**：逐条处理搜索结果，以及它的「回连驱动」接线法
4. **正则切片**：把 LLM 输出的 markdown 报告切成 8 个结构化字段
5. **十大实锤坑**：本模板是我们迄今解剖过「最能跑但也最脆」的模板（见 docs/02-pitfalls.md）

## 课程文件

| 文件 | 内容 |
|---|---|
| `workflow.json` | 官方模板原样（12 功能节点） |
| `workflow-custom.json` | 中国可跑免费替代版（DeepSeek + DBpedia + 本地存档 + 飞书通知，已脱敏） |
| `docs/00~05` | 总览 / 架构 / 坑集 / 验证报告 / 免费替代 / 生产加固 |
| `exercises/exercise.md` | 3 道动手练习（含答案） |

## 三层验证结论（本课全部实测通过）

- **A 单元级**：4 个 Code 节点忠实移植，**30/30 断言全过**
- **B mock 全链**：6 场景（正常/空主题/围栏/脏输出/零结果/格式漂移），**17/17 断言全过**
- **C 真跑**：DeepSeek×2 + DBpedia + 飞书群通知真实 E2E，**加固 prompt 后 8 章节报告齐出**

## 一分钟看懂它干嘛

```text
POST {"topic":"honey production"}
  → AI 拆出 5 个搜索查询
  → 逐条查百科、抓正文
  → AI 综合成 8 章节报告（摘要/发现/事实/益处/挑战/趋势/展望/参考）
  → 存 Google Sheets + Slack 通知 + 回传 JSON
```