# Crossref API 参考

基础 URL: `https://api.crossref.org` | 无需认证 | 礼貌参数: `mailto=<email>`

Crossref 是 DOI 注册机构，提供出版物元数据、期刊信息、出版商等结构化信息。**免费、无 key、无需注册**。

## 认证

Crossref 完全免费开放，无需 API key。建议加 `mailto=<email>` 参数用于礼貌访问和联系。

## Works — 文献查询

### 按 DOI 精确查询

```
GET /works/<DOI>
```

示例: `https://api.crossref.org/works/10.1038/nature12373`

返回完整元数据：标题、作者、期刊、卷期页码、摘要、引用数、许可证、出版日期等。

### 关键词搜索

```
GET /works?query=<关键词>&rows=20&offset=0
```

| 参数 | 说明 |
|------|------|
| `query` | 搜索词，支持布尔（AND/OR/NOT）、短语 `"exact phrase"` |
| `rows` | 每页结果数，最大 1000 |
| `offset` | 分页偏移 |
| `filter` | 过滤条件（见下方） |
| `sort` | `relevance`（默认）/ `published` / `updated` / `deposited` |
| `select` | 返回字段过滤，如 `DOI,title,author,published` |

### Filter 常用字段

| 字段 | 示例 |
|------|------|
| `type` | `type:journal-article`（`book`, `proceedings`, `dissertation`, `report` 等） |
| `from-pub-date` | `from-pub-date:2024-01-01` |
| `until-pub-date` | `until-pub-date:2024-12-31` |
| `has-license` | `has-license:true` |
| `has-full-text` | `has-full-text:true` |
| `has-references` | `has-references:true` |
| `has-abstract` | `has-abstract:true` |
| `issn` | `issn:0028-0836` |
| `publisher` | `publisher:Nature` |
| `member` | `member:297`（出版商 member ID） |
| `funder` | `funder:10.13039/100000001`（按基金项目） |

多个 filter 用逗号（AND）：`filter=type:journal-article,from-pub-date:2024-01-01`

## Journals — 期刊查询

```
GET /journals/<ISSN>
```

返回期刊标题、出版商、ISSN、覆盖范围等元数据。

## Members — 出版商查询

```
GET /members/<member_id>
```

返回出版商名称、旗下期刊数、DOI 前缀等。

## Funders — 基金查询

```
GET /funders/<funder_id>
```

返回基金机构名称、国家等。

## 响应格式

```json
{
  "status": "ok",
  "message-type": "work-list",
  "message-version": "1.0.0",
  "message": {
    "items": [
      {
        "DOI": "10.xxx/xxx",
        "title": ["Paper Title"],
        "author": [
          {"given": "...", "family": "...", "ORCID": "..."}
        ],
        "publisher": "...",
        "container-title": ["Nature"],
        "volume": "612",
        "issue": "7941",
        "page": "123-130",
        "published": {"date-parts": [[2024, 6, 1]]},
        "abstract": "...",
        "is-referenced-by-count": 42,
        "link": [{"URL": "...", "content-type": "text/html"}],
        "license": [{"URL": "...", "start": {...}}]
      }
    ],
    "total-results": 12345,
    "items-per-page": 20,
    "query": { ... }
  }
}
```

## 搜索语法

| 语法 | 示例 |
|------|------|
| 精确短语 | `query="deep learning"` |
| 布尔 | `query=transformer+AND+attention` |
| 排除 | `query=neural+NOT+review` |
| 作者 | `query=author:Hinton` |
| 标题 | `query=title:transformer` |
| 组合 | `query="neural network"+AND+title:attention` |

## 速率

- 非商业使用友好，一般不会被限制
- 批量化请求建议加 `mailto` 参数
