# Brave Search API 参考

基础 URL: `https://api.search.brave.com/res/v1` | 认证: Header `X-Subscription-Token` | Accept: `application/json`

Brave Search 是独立搜索引擎的 API，特色是 LLM 优化上下文（RAG 片段）、无关结果少、隐私友好。科研场景适合找最新网页资料、技术博客、新闻。

## 认证

遵循 [统一密钥管理](../SKILL.md#认证与密钥) 流程。密钥文件: `~/.restack/brave_key`。Brave **必须** API key。

### 读取逻辑

1. 读取 `~/.restack/brave_key`
2. 文件不存在 → AskUserQuestion:
   - "需要 Brave Search API key，去 https://brave.com/search/api/ 申请（注册即赠 $5/月免费额度）"
   - 选项 A: "输入 key" → 写入 `~/.restack/brave_key`
   - 选项 B: "不需要" → 写入 `no`，以后跳过 Brave
3. 内容为 `no` → 跳过，不做 Brave 搜索
4. 内容为有效 key → 请求头加 `X-Subscription-Token: <key>`，再加 `Accept: application/json`
5. 返回 401/429 → 提示 key 无效或超额

## 端点

| 端点 | 用途 |
|------|------|
| `GET /web/search` | 网页搜索 |
| `GET /llm/context` | LLM 优化片段（RAG 场景首选） |
| `GET /news/search` | 新闻搜索 |
| `GET /images/search` | 图片搜索 |
| `GET /videos/search` | 视频搜索 |
| `GET /suggest/search` | 搜索补全 |
| `GET /spellcheck/search` | 拼写纠正 |
| `POST /chat/completions` | 带引用的对话（OpenAI SDK 兼容） |

## Web Search

```
GET /web/search?q=<关键词>&count=20&country=us&search_lang=en
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `q` | string | **必填**，搜索词 |
| `count` | int | 结果数（最大 20） |
| `country` | string | 国家代码，如 `us`, `cn` |
| `search_lang` | string | 语言，如 `en`, `zh` |
| `spellcheck` | int | `1` 启用拼写纠正 |

### 响应

```json
{
  "type": "search",
  "web": {
    "results": [
      {
        "title": "...",
        "url": "...",
        "description": "内容摘要",
        "profile": { "name": "来源名", "img": "favicon" }
      }
    ]
  }
}
```

## LLM Context — RAG 优化片段

专门为 LLM 设计的端点，返回更适合 RAG 的片段，每条结果最多 5 个 snippet。

```
GET /llm/context?q=<关键词>
```

```json
{
  "grounding": {
    "generic": [
      {
        "url": "...",
        "title": "...",
        "snippets": ["片段1", "片段2", "..."]
      }
    ]
  },
  "sources": {
    "<url>": {
      "title": "...",
      "hostname": "...",
      "age": "2024-06-01"
    }
  }
}
```

科研场景适用：搜到资料后，用 LLM Context 获取可直接喂给 LLM 的片段。

## News Search

```
GET /news/search?q=<关键词>&count=10&country=us&search_lang=en&spellcheck=1
```

返回最新新闻，含 `title`, `url`, `description`, `age`, `thumbnail`。

## Chat Completions — 带引用的对话

OpenAI SDK 兼容，返回带引用的答案。

```
POST /chat/completions
Content-Type: application/json
X-Subscription-Token: <key>

{
  "model": "brave",
  "messages": [
    {"role": "user", "content": "What is the latest in quantum computing?"}
  ],
  "stream": false
}
```

## 速率与定价

| 计划 | 速率 | 价格 |
|------|------|------|
| Search | 50 QPS | $5/1000 请求（每月赠 $5） |
| Answers (Chat) | 2 QPS | $4/1000 请求 + $5/百万 token |

## 科研搜索最佳实践

1. **找最新资料** → News Search + `search_lang: "en"`
2. **给 LLM 喂上下文** → LLM Context 端点，返回即用的 snippet
3. **需要带引用的答案** → Chat Completions（有 citation）
4. **常规搜索** → Web Search，结果比通用搜索引擎更干净
