# 05 · 生产加固指南

按优先级排序，抄走即用：

## P0 · 不修不能上生产

1. **循环瘦身**：把 splitInBatches 的回连点从末端改到 `Clean Website Content`；Merge 换「组合聚合」模式收齐所有词条；AI 综合/存档/通知/响应只跑一次。（修坑1，费用 ÷N，通知 1 条）
2. **prompt 锁格式**：AI Research Engine 的 prompt 加一行：「Format every section heading exactly as a markdown level-3 heading with the ### prefix」。（修坑5）
3. **Parse 加兜底**：try/catch + 正则提取 `[...]` 数组，失败时抛带原文片段的错误。（修坑4）

## P1 · 数据质量

4. Prepare 改 `$input.all()` 展平全部查询的结果（修坑2）
5. Formatter 的 topic 改从 `Receive Research Topic` 取（修坑3）
6. Clean 后加 If 过滤空 content（修坑6）
7. content `.slice(0, 12000)` 截断（修坑8）

## P2 · 可观测性

8. Validate false 分支接 respondToWebhook 返回 400 + 原因（修坑7）
9. 零搜索结果时回传「无资料」而不是静默结束
10. Sheets 存档加「查询原文」「词条数」「耗时」列；通知里带摘要前 200 字

## 改完的预期形态

```text
1 次请求 → 5 查询全搜 → N 词条聚合 → 1 次 AI 综合 → 1 行存档 → 1 条通知 → 1 次完整响应
```