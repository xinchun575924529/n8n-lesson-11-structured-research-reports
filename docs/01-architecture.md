# 01 · 架构与数据流

## 数据流图

```text
Webhook ──→ Set(topic) ──→ If(非空) ──→ OpenAI#1(造5查询)
   ──→ Parse(JSON.parse) ──→ Search(Wikipedia×N) ──→ Prepare(清洗)
   ──→ splitInBatches ─┬─(loop)→ Extract(抓正文) → Clean → Merge
                       │        → OpenAI#2(综合) → Formatter(切8段)
                       │        → Sheets(存档) → Slack(通知) → Respond ─┐
                       └────────────────────────────────────────────────┘
```

## ⚠️ 看懂这张图的关键：循环体里装了什么

按直觉，「AI 综合 → 存档 → 通知」应该在循环**结束**后跑一次。但本模板的循环回连点在 **Return Research Response**（最后一个节点）——意味着：

- **循环体 = 取正文 + 清洗 + 合并 + AI 综合 + 写表 + 通知 + 响应** 一整条尾巴
- 每条搜索结果都会触发**一次完整的报告生成**
- 5 查询 × 5 结果 = 最多 **25 次 AI 综合 + 25 条 Slack 消息 + 25 次 webhook 响应**（只有第一次响应真的回到调用方）

这不是我们猜的——B 关 mock 全链实锤（2 查询 × 2 结果 → 2 次 AI 综合、2 条通知、2 次响应），C 关真跑复现（5 次写表、5 条飞书通知）。

## 另一个反直觉点：只有第一个查询被真正检索

`Prepare Search Results` 是 Code 节点（默认 runOnceForAllItems 模式），里面写的是 `$input.first().json.query.search`——**只取第一条输入的搜索结果**。OpenAI 辛苦造的 5 个查询，4 个白搜了。B 关实锤：第二个查询（trends）的结果从未进入报告。

## Merge 节点在这里是摆设

`Merge Research Data` 空参数、单输入——它什么都没有「合并」，只是直通。作者大概本想用它聚合循环产物，但接线方式决定了它每轮只见到 1 个 item。