# arXiv 预印本参考

arXiv 覆盖物理学、数学、计算机科学（AI/CV/NLP/ML）、生物学、经济学等领域的最新预印本。**通过 MCP 工具调用，无需 API key。**

## 认证

无需认证，通过 arXiv MCP 工具直接调用。不需要 API key。

## MCP 工具

### search_papers — 搜索论文

```
mcp__arxiv-mcp-server__search_papers
  query: "关键词" (必填)
  max_results: 20 (默认 10，最多 50)
  categories: ["cs.AI", "cs.CL"] (强烈建议)
  sort_by: "relevance" | "date"
  date_from: "2024-01-01"
  date_to: "2024-12-31"
```

**查询技巧**：
- 用引号短语：`"multi-agent systems"`
- 用 OR 组合：`"AI agents" OR "software agents"`
- 用 ANDNOT 排除：`"deep learning" ANDNOT "survey"`
- 限定标题：`ti:"transformer architecture"`
- 限定作者：`au:"Hinton"`

### get_abstract — 获取摘要

```
mcp__arxiv-mcp-server__get_abstract
  paper_id: "2401.12345"
```

返回标题、作者、摘要、分类、发表日期、PDF 链接。在 download 前用这个判断相关性。

### download_paper — 下载全文

```
mcp__arxiv-mcp-server__download_paper
  paper_id: "2401.12345"
```

先尝试 HTML 版本（干净提取），失败则 PDF 转换。

### read_paper — 阅读已下载论文

```
mcp__arxiv-mcp-server__read_paper
  paper_id: "2401.12345"
```

返回 Markdown 格式全文。必须先 download 才能 read。

### citation_graph — 查看引用关系

```
mcp__arxiv-mcp-server__citation_graph
  paper_id: "2401.12345"
```

返回引用该论文的论文列表和该论文引用的论文列表。数据来自 Semantic Scholar。

### semantic_search — 语义搜索

```
mcp__arxiv-mcp-server__semantic_search
  query: "attention mechanisms for long sequences"
  paper_id: "2401.12345" (可选，找相似论文)
  max_results: 10
```

只搜索**已下载到本地**的论文。需要先 download 论文才能被索引。

## 常用分类

| 代码 | 领域 |
|------|------|
| cs.AI | 人工智能 |
| cs.LG | 机器学习 |
| cs.CL | 计算语言学 / NLP |
| cs.CV | 计算机视觉 |
| cs.MA | 多智能体系统 |
| cs.NE | 神经与进化计算 |
| cs.RO | 机器人学 |
| cs.IR | 信息检索 |
| cs.CR | 密码学与安全 |
| stat.ML | 统计 — 机器学习 |
| math.OC | 数学 — 优化与控制 |
| quant-ph | 量子物理 |
| q-bio | 定量生物学 |

## 典型工作流

1. `search_papers` → 关键词 + 分类找到 20-50 篇
2. `get_abstract` → 快速筛选，保留 5-10 篇相关
3. `download_paper` → 下载选中的论文
4. `read_paper` → 深度阅读全文
5. `citation_graph` → 展开引用链
6. `semantic_search` → 在本地找相关论文
