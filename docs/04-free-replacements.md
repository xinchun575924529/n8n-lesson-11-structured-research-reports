# 04 · 免费替代方案（中国可跑版）

| 模板需求 | 原版 | 免费替代 | 实测 |
|---|---|---|---|
| LLM ×2 | OpenAI gpt-4o-mini（付费） | **DeepSeek deepseek-chat** 直连 | ✅ 单次全链几分钱 |
| 检索源 | Wikipedia API | **DBpedia lookup**（可达/免费/无 key） | ✅ 可用但内容偏浅* |
| 存档 | Google Sheets OAuth | **本地 JSON** 或飞书多维表格 | ✅ |
| 通知 | Slack | **飞书群**（tenant token 直发） | ✅ notifyCode=0 |

\* DBpedia 公共端点数据集被裁剪（无 abstract 谓词），正文只能用 lookup 的 comment 字段；检索相关性也偏弱。要更深的正文：把检索层换成你自己可及的源（企业内网知识库 / 海外节点 / 自建代理），工作流结构不变。

## 替换口径（workflow-custom.json 已做好）

1. 两个 OpenAI 节点 → httpRequest POST `https://api.deepseek.com/chat/completions`（前置 Code 预构 `reqBody` JSON 字符串，见坑10）
2. Wikipedia 两个 httpRequest → DBpedia lookup（后跟 2 个形状适配 Code，把 DBpedia 响应整成 Wikipedia 形状——**适配器模式**，下游节点零改动）
3. Sheets → Code 写本地 JSON（appendOrUpdate 语义复刻）
4. Slack → Code 调飞书 OpenAPI（先取 tenant_access_token 再发消息）

## 适配器模式（本课隐藏知识点）

换数据源时**不要**改下游业务节点，而是在新源后面插一个「形状适配」Code 节点，把响应整成旧源的数据形状。19710 里 Prepare/Clean 节点因此一行没改。