# Semantic Scholar API 参考

基础 URL: `https://api.semanticscholar.org/graph/v1` | 推荐 API: `https://api.semanticscholar.org/recommendations/v1`

## 认证

遵循 [统一密钥管理](../SKILL.md#认证与密钥) 流程。密钥文件: `~/.restack/s2_key`。

### 读取逻辑

1. 读取 `~/.restack/s2_key`
2. 文件不存在 → AskUserQuestion:
   - "需要 Semantic Scholar API key，去 https://www.semanticscholar.org/product/api#api-key 免费申请"
   - 选项 A: "输入 key" → 写入 `~/.restack/s2_key`
   - 选项 B: "不需要" → 写入 `no`，以后匿名访问
3. 内容为 `no` → 匿名访问（共享 1000 req/s 池）
4. 内容为有效 key → 请求头加 `x-api-key: <key>`
5. 返回 429 → 提示超速率或 key 无效

### 额度

- 匿名：所有用户共享 1000 req/s，高峰期额外限流
- 有 key：独立 1 req/s（所有端点合计），审核后可申请更高

## 论文搜索

### 批量搜索（推荐，适合大多数场景）

```
GET /paper/search/bulk?query=<关键词>&sort=citationCount:desc&limit=100
```

| 参数 | 说明 |
|------|------|
| `query` | 搜索词（必填），支持高级语法 |
| `token` | 分页游标，从响应 `next` 字段获取 |
| `sort` | `paperId`, `publicationDate`, `citationCount`（默认 `paperId`） |
| `publicationTypes` | 过滤类型 |
| `openAccessPdf` | 是否含公开 PDF |
| `minCitationCount` | 最低引用数 |
| `publicationDateOrYear` | 日期范围，如 `"2023-01-01:2024-12-31"` |
| `year` | 年份范围，如 `"2023-"`（2023 至今） |
| `venue` | 出版 venue 过滤 |
| `fieldsOfStudy` | 研究领域过滤 |

### 相关性搜索（更精确，需更多资源）

```
GET /paper/search?query=<关键词>&limit=20&offset=0
```

参数: `query`（必填）, `limit`, `offset`, `fields`

### 搜索语法

| 语法 | 示例 | 说明 |
|------|------|------|
| 短语 | `"generative ai"` | 精确匹配 |
| 必需 | `+security` | 必须包含 |
| 排除 | `-privacy` | 排除该词 |
| 或 | `cloud\|virtualization` | 匹配任一 |
| 通配符 | `fish*` | 前缀匹配 |
| 模糊 | `bugs~3` | 编辑距离 ≤ N |
| 间隔 | `"blue lake"~3` | 词间最多 N 个词 |

匹配范围: title + abstract。

## 论文详情

### 单篇

```
GET /paper/{paper_id}?fields=title,year,authors,abstract,citationCount,journal,url
```

`paper_id` 支持: Semantic Scholar ID、DOI、arXiv ID、Corpus ID、PubMed ID。

### 批量

```
POST /paper/batch?fields=title,year,citationCount
{"ids": ["id1", "id2", "DOI:10.xxx", "ARXIV:2103.xxx"]}
```

## 引用关系

### 引用（谁引了这篇）

```
GET /paper/{paper_id}/citations?limit=50&fields=title,year,citationCount
```

### 参考文献（这篇引了谁）

```
GET /paper/{paper_id}/references?limit=50&fields=title,year,citationCount
```

## 作者

### 单作者

```
GET /author/{author_id}?fields=name,paperCount,hIndex,affiliations,papers
```

### 批量

```
POST /author/batch?fields=name,paperCount,hIndex
{"ids": ["authorId1", "authorId2"]}
```

## 推荐

### 基于单篇

```
GET https://api.semanticscholar.org/recommendations/v1/papers?seed_paper_id={paper_id}&limit=20
```

### 基于正负列表

```
POST https://api.semanticscholar.org/recommendations/v1/papers?limit=20
{"positivePaperIds": ["id1","id2"], "negativePaperIds": ["id3"]}
```

limit 最大 500，结果按相关性降序。

## 常用 fields 参数

所有端点通过 `&fields=` 控制返回字段，逗号分隔：

```
title,year,authors,abstract,externalIds,url,venue,journal,citationCount,influentialCitationCount,referenceCount,publicationDate,openAccessPdf,fieldsOfStudy,publicationTypes,embedding,tldr
```

- `authors` 含 `authorId` 和 `name`
- `externalIds` 含 `DOI`, `ArXiv`, `PubMed`, `CorpusId`
- `openAccessPdf` 含公开 PDF URL
- `tldr` 含 AI 生成的一句话摘要
- `embedding` 含 SPECTER2 向量

## 响应格式

```json
{
  "total": 12345,
  "offset": 0,
  "next": 100,
  "data": [ ... ]
}
```

bulk search 用 `token` 分页（不是 offset），从响应中取 `next` 作为下页 token：

```
GET /paper/search/bulk?query=...&token=<next_token>
```

## HTTP 状态码

| 码 | 含义 |
|----|------|
| 200 | 成功 |
| 400 | 参数错误 |
| 401 | 认证失败 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 429 | 超速率限制 |
| 500 | 服务器错误 |
