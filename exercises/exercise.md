# 练习（含答案）

## 练习 1 · 找出报告的「真 Topic」

不改任何节点类型，只改一个表达式，让 Google Sheets 的 Topic 列存**研究主题**而不是词条标题。写出改哪个节点、哪个字段、改成什么。

<details><summary>答案</summary>

改 `Research Report Formatter` 的 `topic` 字段：

```js
// 原（错）：$('Merge Research Data').first().json.title   ← 词条标题
topic: $('Receive Research Topic').first().json.topic
```
</details>

## 练习 2 · 修好循环

当前结构下 25 条搜索结果会产生 25 次 AI 综合。画出修复后的接线：哪个节点的输出回连到 splitInBatches？尾巴（Merge→AI→Formatter→Sheets→Slack→Respond）挂在哪里？

<details><summary>答案</summary>

- 回连点：`Clean Website Content` → `Process Search Results`（循环体只剩 Extract→Clean）
- splitInBatches 的 **done 输出**（main[0]，模板里一直空挂）→ `Merge Research Data` → 尾巴各节点
- Merge 设 combine/aggregate 语义收齐全部词条为一个数组，AI prompt 里 `JSON.stringify` 注入
</details>

## 练习 3 · 让 Parse 永不崩

给 `Parse Search Queries` 写加固版：LLM 返回带开场白、带围栏、甚至数组外有多余文字时都能解析；完全无法解析时抛出包含原文前 100 字符的错误。

<details><summary>答案</summary>

```js
const text = $json.output?.[0]?.content?.[0]?.text || '';
const m = text.match(/\[[\s\S]*\]/);
if (!m) throw new Error('LLM 未返回 JSON 数组: ' + text.slice(0, 100));
let queries;
try { queries = JSON.parse(m[0]); }
catch (e) { throw new Error('JSON 解析失败: ' + e.message + ' | 原文: ' + text.slice(0, 100)); }
return queries.map(query => ({ json: { query } }));
```
</details>