---
name: research
description: 多源学术文献搜索 — 整合 OpenAlex、Semantic Scholar、Crossref、arXiv、Tavily、Brave 进行全方位文献检索
---

# 学术文献搜索

执行学术搜索请求时，根据场景选择数据源组合，输出结构化结果。

## 认证与密钥

所有数据源统一使用 `~/.restack/<name>_key` 管理 API key。每次调用前按以下流程处理：

1. 读取 `~/.restack/<name>_key`
2. **文件不存在** → 使用 AskUserQuestion 询问：
   - 选项 A："输入 key" → 用户粘贴后写入 `~/.restack/<name>_key`
   - 选项 B："不需要" → 写入 `no` 到 `~/.restack/<name>_key`，以后跳过
   - 提问时给出申请 URL 和用途说明
3. **文件内容为 `no`** → 匿名访问或跳过该源
4. **文件内容为有效 key** → 使用该 key 发起请求
5. **返回 4xx 认证错误** → 提示 key 无效，建议去申请 URL 重新获取

密钥写入：`mkdir -p ~/.restack && echo "YOUR_KEY" > ~/.restack/<name>_key`

## 数据源总览

| 数据源 | 定位 | 是否需要 key | 调用方式 | 参考 |
|--------|------|:----------:|---------|------|
| OpenAlex | 文献主索引（2.5 亿+） | 可选 | WebFetch → REST | [references/openalex.md](references/openalex.md) |
| Semantic Scholar | 引用关系、相关推荐 | 可选 | WebFetch → REST | [references/semantic-scholar.md](references/semantic-scholar.md) |
| Crossref | DOI 查询、出版元数据 | 否 | WebFetch → REST | [references/crossref.md](references/crossref.md) |
| arXiv | AI/CV/NLP/物理 最新预印本 | 否 | MCP 工具 | [references/arxiv.md](references/arxiv.md) |
| Tavily | AI 网络搜索 | 必须 | WebFetch → REST | [references/tavily.md](references/tavily.md) |
| Brave | 独立搜索引擎、LLM 片段 | 必须 | WebFetch → REST | [references/brave.md](references/brave.md) |

## 搜索策略

### 第一步：问题分类

收到搜索请求后，先判断问题类型，再选择对应的数据源链路。**不要把所有的需求都给同一个 API。**

| 问题类型 | 示例 | 数据源链路（按优先级） |
|----------|------|----------------------|
| 文献类 | 论文搜索、领域综述、作者调查、引用分析 | OpenAlex → Semantic Scholar → Tavily → Brave |
| 技术文档类 | API 用法、框架文档、代码示例、部署方案 | Tavily → Brave |
| 普通网页类 | 新闻事件、产品信息、人物背景、百科知识 | Tavily → Brave |

### 第二步：Query Rewrite（复杂问题必须执行）

对于多面问题、跨领域问题或模糊问题，先拆解为 2-4 个子查询，分别检索。

- 模糊问题 → 提取核心概念，生成精确短语查询
- 多面问题 → 每个面一个独立查询（如"Transformer 在 CV 和 NLP 中的应用"拆为两个查询）
- 跨语言 → 中英文各跑一次

### 第三步：多源检索

按问题类型对应的链路顺序执行：

- **OpenAlex**（文献主索引）→ 按关键词 + 主题 + 年份大面积检索，获取文献 ID、DOI、引用量
- **Semantic Scholar**（引用与摘要）→ 用 OpenAlex 返回的 ID/DOI 获取引用关系、相关论文推荐、摘要；或用搜索端点直接搜
- **Tavily**（正文证据）→ agent-friendly 网页搜索，获取正文片段、博客、技术文档、最新报道
- **Brave**（通用 fallback）→ 独立搜索引擎，补充 Tavily 覆盖面不足的部分；LLM Context 端点获取 RAG 片段

单次简单搜索走一个链路即可；复杂搜索（如写综述）需要走完整的多源链路。

### 第四步：去重、Rerank 与来源校验

1. **去重** — 按 DOI 合并 OpenAlex 和 Semantic Scholar 的结果；按 URL 合并 Tavily 和 Brave 的网页结果
2. **Rerank** — 按（引用量降序 + recency 加权 + 来源权威性）重新排序
3. **来源校验** — 优先保留有 DOI 的论文、`.edu` / `.org` 域名的网页、知名出版商来源；标记预印本属性

### 各源分工

| 数据源 | 角色 | 擅长 | 不擅长 |
|--------|------|------|--------|
| OpenAlex | 文献主索引 | 按主题/作者/机构/年份全覆盖检索，DOI 解析 | 最新预印本可能有延迟 |
| Semantic Scholar | 引用补充 | 引用图、相关推荐、NLP 增强搜索、影响力评分 | 不覆盖非学术来源 |
| Tavily | 正文证据 | agent-friendly 搜索、正文提取、最新网页 | 非学术数据库 |
| Brave | 通用 fallback | 独立索引、LLM 优化片段、新闻搜索 | 无学术结构化元数据 |
| Crossref | 元数据校验 | DOI 确认、期刊信息、卷期页码 | 搜索能力弱 |
| arXiv | 预印本 | 最新 AI/CV/NLP 论文、全文下载与阅读 | 仅预印本，无同行评审 |

## 输出格式

每条结果必须保留以下证据字段：

```json
{
  "title": "论文标题或网页标题",
  "url": "访问链接",
  "source_api": "openalex | semantic_scholar | crossref | arxiv | tavily | brave",
  "source_type": "paper | preprint | article | blog | doc | news | other",
  "year": 2024,
  "doi": "10.xxx/xxx（如有）",
  "authors": ["Author Name"],
  "abstract_or_snippet": "摘要或正文片段（切勿编造）",
  "retrieved_at": "2026-06-02T12:00:00+08:00"
}
```

汇总时先给表格一览，再分条给证据详情：

| # | 标题 | 作者 | 年 | 来源 | 类型 | 引用 | 链接 |
|---|------|------|----|------|------|------|------|
| 1 | ... | ... | 2024 | openalex | paper | 42 | [链接](url) |

## 禁止行为

- **禁止编造**: 来源、DOI、作者名、年份、引用量、价格、版本号、下载量等字段必须来自 API 返回值。证据不足时明确说"未找到/未确认"，不要补全
- **禁止单一源**: 复杂问题不得仅依赖一个搜索 API；如果某个源无结果，换下一个源
- **禁止省略证据**: 每条结果必须带 `source_api` 和 `retrieved_at`
- **禁止盲信预印本**: arXiv 结果需标注为 `preprint`，提醒用户未经同行评审
