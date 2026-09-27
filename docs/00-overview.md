# 00 · 模板总览

## 定位

「AI 个人研究助理」：把人工上网查资料、写调研报告的过程自动化。官方场景是给工程/研究团队用：调一个 webhook，几分钟后在 Slack 收到完成通知，报告躺在 Google Sheets 里。

## 节点清单（12 功能节点）

| 节点 | 类型 | 职责 |
|---|---|---|
| Webhook | webhook | POST /research-assistant，responseNode 双向模式 |
| Receive Research Topic | set | 提取 body.topic |
| Validate Research Topic | if | topic 非空校验 |
| z（对，名字就叫 z） | langchain.openAi | gpt-4o-mini 生成 5 个搜索查询 |
| Parse Search Queries | code | 剥 markdown 围栏、JSON.parse |
| Search Web | httpRequest | Wikipedia search API（srlimit=5） |
| Prepare Search Results | code | 清洗 snippet、拼词条 URL |
| Process Search Results | splitInBatches | 逐条循环（batch=1） |
| Extract Website Content | httpRequest | Wikipedia extracts API 抓正文 |
| Clean Website Content | code | 配对原始 item、取 extract |
| Merge Research Data | merge | 聚合（空参数=append 直通） |
| AI Research Engine | langchain.openAi | gpt-4o-mini 综合成 8 章节报告 |
| Research Report Formatter | code | 正则按 ### 切 8 段 |
| Update row in sheet | googleSheets | appendOrUpdate 按 Topic 匹配 |
| Send a message | slack | 完成通知 |
| Return Research Response | respondToWebhook | 回传调用方 |

## 适合谁

- 学完 lesson-01~08 想进「多 LLM 编排 + 循环」的中级学员
- 需要给团队做自动化调研/竞品情报流水线的运营