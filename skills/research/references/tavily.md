# Tavily API 参考

基础 URL: `https://api.tavily.com` | 认证: `Authorization: Bearer tvly-YOUR_API_KEY`

Tavily 是专为 AI Agent 优化的搜索 API，适合查找最新博客、技术报告、新闻等非学术来源。

## 认证

遵循 [统一密钥管理](../SKILL.md#认证与密钥) 流程。密钥文件: `~/.restack/tavily_key`。Tavily **必须** API key。

### 读取逻辑

1. 读取 `~/.restack/tavily_key`
2. 文件不存在 → AskUserQuestion:
   - "需要 Tavily API key，去 https://app.tavily.com 申请（有免费额度）"
   - 选项 A: "输入 key" → 写入 `~/.restack/tavily_key`
   - 选项 B: "不需要" → 写入 `no`，以后跳过 Tavily
3. 内容为 `no` → 跳过，不做 Tavily 搜索
4. 内容为有效 key → 请求头加 `Authorization: Bearer <key>`（key 需带 `tvly-` 前缀）
5. 返回 401/429/432/433 → 提示 key 无效或超额

## 端点

| 端点 | 用途 |
|------|------|
| `POST /search` | 网络搜索（核心） |
| `POST /extract` | 网页内容提取 |
| `POST /crawl` | 智能站点爬取 |
| `POST /map` | 站点结构发现 |
| `POST /research` | 深度研究（多步搜索+提取） |

## Search — 网络搜索

```
POST /search
Content-Type: application/json
Authorization: Bearer <key>

{
  "query": "...",
  "search_depth": "advanced",
  "max_results": 10,
  "include_answer": "advanced"
}
```

### 参数

| 参数 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `query` | string | **必填** | 搜索词 |
| `search_depth` | enum | `basic` | `basic`(1 积分) / `advanced`(2 积分，含片段) |
| `max_results` | int | 5 | 0–20 |
| `topic` | enum | `general` | `general` / `news` / `finance` |
| `time_range` | enum | — | `day`, `week`, `month`, `year` |
| `start_date` / `end_date` | string | — | `YYYY-MM-DD` 格式 |
| `include_answer` | bool/string | false | `true`=简要 / `"advanced"`=详细 LLM 答案 |
| `include_raw_content` | bool/string | false | `true` / `"markdown"` / `"text"` |
| `include_images` | bool | false | 返回图片 URL |
| `include_domains` | array | — | 限定的域名（最多 300） |
| `exclude_domains` | array | — | 排除的域名（最多 150） |
| `country` | enum | — | 偏向某国结果（仅 `general` topic） |
| `exact_match` | bool | false | 只返回含精确短语的结果 |

### 响应

```json
{
  "query": "...",
  "answer": "LLM 生成的答案（可选）",
  "results": [
    {
      "title": "...",
      "url": "...",
      "content": "简介/片段",
      "score": 0.95,
      "raw_content": "清洗后的全文（可选）"
    }
  ],
  "images": [],
  "response_time": 1.23,
  "request_id": "..."
}
```

## Extract — 网页提取

提取任意 URL 的全文内容，适合阅读搜索结果中的具体文章。

```
POST /extract
Content-Type: application/json
Authorization: Bearer <key>

{
  "urls": ["https://example.com/article"],
  "extract_depth": "advanced",
  "format": "markdown",
  "query": "用户意图（可选，用于内容重排序）"
}
```

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `urls` | string/array | **必填**，最多 20 个 URL |
| `extract_depth` | enum | `basic`(1 积分/5 URL) / `advanced`(2 积分/5 URL) |
| `format` | enum | `markdown` / `text` |
| `query` | string | 用户意图，用于把最相关的 chunk 排到前面 |
| `chunks_per_source` | int(1-5) | 每源最大块数，需配合 `query` |
| `include_images` | bool | 是否提取图片 |
| `timeout` | float | 1.0–60.0 秒（basic 默认 10s，advanced 默认 30s） |

### 响应

```json
{
  "results": [
    {
      "url": "...",
      "raw_content": "Markdown 格式全文"
    }
  ],
  "failed_results": [],
  "response_time": 2.5,
  "request_id": "..."
}
```

## 科研搜索最佳实践

1. **找最新进展** — `topic: "news"` + `time_range: "week"` + `search_depth: "advanced"`
2. **深度了解主题** — `include_answer: "advanced"` 获取 LLM 总结
3. **阅读原文** — 用 Extract 提取搜索结果中的 URL
4. **排除干扰** — 用 `include_domains` 限定 `.edu`, `.org` 等学术域名
