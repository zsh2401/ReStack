# OpenAlex API 参考

基础 URL: `https://api.openalex.org` | 礼貌参数: `mailto=<email>`

## 认证

遵循 [统一密钥管理](../SKILL.md#认证与密钥) 流程。密钥文件: `~/.restack/openalex_key`。

### 读取逻辑

1. 读取 `~/.restack/openalex_key`
2. 文件不存在 → AskUserQuestion:
   - "需要 OpenAlex API key，去 https://openalex.org/settings/api 免费申请（每天 $1 额度）"
   - 选项 A: "输入 key" → 写入 `~/.restack/openalex_key`
   - 选项 B: "不需要" → 写入 `no`，以后匿名访问
3. 内容为 `no` → 匿名访问
4. 内容为有效 key → 请求时加 `?api_key=<key>`
5. 返回 403 → 提示超额或 key 无效，建议重新申请

### 额度

- 匿名：速率限制严格，适合少量试探查询
- 免费 key：每天 $1 额度，正常学术搜索完全够用

## 实体端点

| 实体 | 列表端点 | 单体端点 |
|------|---------|---------|
| Works（文献） | `/works` | `/works/{id}` |
| Authors（作者） | `/authors` | `/authors/{id}` |
| Sources（来源） | `/sources` | `/sources/{id}` |
| Institutions（机构） | `/institutions` | `/institutions/{id}` |
| Topics（主题） | `/topics` | `/topics/{id}` |
| Keywords | `/keywords` | `/keywords/{id}` |
| Publishers | `/publishers` | `/publishers/{id}` |
| Funders（基金） | `/funders` | `/funders/{id}` |
| Autocomplete | `/autocomplete/{entity}?q=<query>` | — |

## 通用参数

| 参数 | 说明 |
|------|------|
| `filter` | 按字段值过滤（核心参数），多个条件用逗号（AND）或竖线（OR）组合 |
| `search` | 全文搜索（标题+摘要），支持布尔运算、短语搜索 |
| `sort` | 排序，格式 `字段:方向`（如 `cited_by_count:desc`），多个排序用逗号 |
| `per_page` | 每页结果数，默认 25，最大 200 |
| `page` | 页码（数字分页），从 1 开始 |
| `cursor` | 游标分页（深分页），值用 `*` 获取第一页，后续用响应中的 `next_cursor` |
| `sample` | 随机采样 N 条，`&sample=100&seed=42` |
| `select` | 限制返回字段，逗号分隔（如 `id,title,doi`） |
| `group_by` | 按字段聚合，如 `group_by=topics.id` |

**注意**: `cursor` 和 `page` 不可同时使用。推荐用 `cursor` 做深分页。

## 响应格式

```json
{
  "meta": {
    "count": 12345,         // 总结果数
    "db_response_time_ms": 42,
    "page": 1,              // 当前页（page 模式）
    "per_page": 25,
    "next_cursor": "abc123" // 下一页游标（cursor 模式）
  },
  "results": [ ... ]
}
```

## Works Filter — 常用字段

### 时间

| 字段 | 示例 |
|------|------|
| `publication_year` | `publication_year:2024` 或 `publication_year:2020-2024`（范围） |
| `publication_date` | `publication_date:2024-06-01` |
| `from_publication_date` | `from_publication_date:2023-01-01` |
| `to_publication_date` | `to_publication_date:2024-12-31` |
| `from_created_date` | `from_created_date:2024-01-01`（收录时间） |

### 文献类型与语言

| 字段 | 可选值 |
|------|--------|
| `type` | `article`, `book`, `book-chapter`, `dataset`, `dissertation`, `preprint`, `report`, `other`, `paratext`, `standard`, `reference-entry`, `peer-review`, `editorial`, `erratum`, `letter`, `grant` |
| `language` | ISO 639-1 代码，如 `en`, `zh`, `de`, `fr` |

### 开放获取

| 字段 | 说明 |
|------|------|
| `open_access.is_oa` | `true` / `false` |
| `open_access.oa_status` | `gold`, `green`, `hybrid`, `bronze`, `closed` |
| `has_fulltext` | `true` / `false` |
| `has_pdf_url` | `true` / `false` |

### 引用与影响力

| 字段 | 示例 |
|------|------|
| `cited_by_count` | `cited_by_count:>100`（支持 `>`, `<`, `>=`, `<=`） |
| `cited_by` | `cited_by:W12345`（被某文献引用） |
| `cites` | `cites:W12345`（引用了某文献） |
| `referenced_works` | `referenced_works:W12345` |

### 作者与机构

| 字段 | 示例 |
|------|------|
| `authorships.author.id` | `authorships.author.id:A5023888391` |
| `authorships.author.orcid` | `authorships.author.orcid:0000-0002-1234-5678` |
| `authorships.institutions.id` | `authorships.institutions.id:I27837315` |
| `authorships.institutions.country_code` | `authorships.institutions.country_code:CN` |
| `authorships.institutions.type` | `education`, `government`, `nonprofit`, `company` 等 |
| `authorships.institutions.continent` | `authorships.institutions.continent:AS` |
| `authorships.is_corresponding` | `true` / `false` |
| `authors_count` | `authors_count:1` 或 `authors_count:>5` |

### 主题与概念

| 字段 | 示例 |
|------|------|
| `topics.id` | `topics.id:T10123` |
| `topics.subfield.id` | `topics.subfield.id:1701547` |
| `primary_topic.id` | `primary_topic.id:T10123` |
| `concepts.id` | `concepts.id:C12345` |
| `keywords.id` | `keywords.id:K12345` |

### 来源与期刊

| 字段 | 示例 |
|------|------|
| `journal` | `journal:Nature` |
| `primary_location.source.id` | `primary_location.source.id:S12345` |
| `primary_location.source.issn` | `primary_location.source.issn:0028-0836` |
| `primary_location.source.type` | `journal`, `repository`, `conference` 等 |

### 标识符

| 字段 | 示例 |
|------|------|
| `doi` | `doi:10.1038/nature12373`（精确匹配） |
| `doi_starts_with` | `doi_starts_with:10.1038/`（前缀匹配） |
| `has_doi` | `true` / `false` |
| `pmid` | `pmid:12345678` |
| `pmcid` | `pmcid:PMC1234567` |
| `ids.mag` | `ids.mag:123456789` |

### 基金与奖项

| 字段 | 示例 |
|------|------|
| `funders.id` | `funders.id:F12345` |
| `awards.funder_id` | `awards.funder_id:F12345` |
| `awards.id` | `awards.id:A12345` |

### 其他常用

| 字段 | 说明 |
|------|------|
| `is_retracted` | 是否撤稿 |
| `has_references` | 是否有参考文献列表 |
| `has_abstract` | 是否有摘要 |
| `has_orcid` | 是否有 ORCID 作者 |
| `has_pmid` | 是否有 PubMed ID |
| `version` | 版本号 |
| `indexed_in` | 索引来源，如 `crossref` |

## Search 全文搜索

使用 `search` 参数（非 filter）进行标题+摘要全文搜索：

```
/works?search=reinforcement+learning&per_page=20
```

- **短语搜索**: `search="deep reinforcement learning"`
- **布尔**: `search=transformer+AND+attention`（AND/OR/NOT）
- **结合 filter**: `search=GAN&filter=publication_year:2024,type:article`

**弃用的 `.search` filters**（不要用）: `title.search`, `abstract.search`, `fulltext.search`, `display_name.search` — 统一用 `search` 参数。

## Sort 排序

常用排序字段：
- `cited_by_count` — 被引量
- `publication_date` — 发表日期
- `publication_year` — 发表年份
- `relevance_score` — 搜索相关性（需要 search 参数）
- `title` — 标题字母序
- `type` — 类型
- `updated_date` — 更新时间
- `authors_count` — 作者数

格式: `sort=cited_by_count:desc`，**asc 或 desc 必选**。多字段: `sort=cited_by_count:desc,publication_date:desc`。

## Group By 聚合

```
/works?filter=publication_year:2024&group_by=topics.id
```

响应中 `groups` 数组替代 `results`，每组含 `key`, `count`, `key_display_name`。

支持 group_by 的常用字段: `publication_year`, `type`, `language`, `oa_status`, `authorships.institutions.id`, `topics.id`, `primary_topic.id`, `journal`, `license`。

## 运算符与组合

| 操作 | 语法 | 示例 |
|------|------|------|
| 等于 | `field:value` | `type:article` |
| 不等于 | `field:!value` | `type:!book` |
| 大于/小于 | `field:>value`, `field:<value` | `cited_by_count:>100` |
| 范围 | `field:v1-v2` | `publication_year:2020-2024` |
| AND | 逗号 `,` | `type:article,language:en` |
| OR | 竖线 `\|` | `type:article\|type:book` |
| 多重 OR | `field:v1\|v2\|v3` | `doi:10.a\|10.b\|10.c` |

AND 优先级高于 OR。

## 速率与用量

- 免费 API key: $1/天 额度
- 礼貌访问: 加 `mailto=<email>` 参数
- 推荐使用 `cursor` 而非 `page` 做深分页

## 获取单个 Work

```
GET /works/W3038568908
```

返回完整 Work 对象，含 abstract、authorships、concepts、topics、cited_by_api_url 等全字段。
