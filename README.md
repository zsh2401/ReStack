# ReStack

> "The scientist is not a person who gives the right answers, he's one who asks the right questions." — Claude Lévi-Strauss

AI agents can write code. They can also do research. But without the right tooling, they default to surface-level web searches that miss the literature that matters. ReStack gives Claude Code a research team: a literature search specialist who knows when to query OpenAlex vs Semantic Scholar vs arXiv, a LaTeX template finder who checks publisher sources before Overleaf, and a document archivist who turns your sessions into structured research notes.

ReStack is built for scientists, PhD students, and anyone doing rigorous literature work. It is free, MIT licensed, and lives entirely in your `~/.claude/skills/` directory.

## Quick start

1. Install ReStack (30 seconds — see below)
2. Type `/research` with any academic search query
3. Type `/latex-template` to find a conference/journal template
4. Type `/docify` to save your session to structured docs
5. Stop there. You'll know if this is for you.

## Install — 30 seconds

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/)

### Step 1: Install on your machine

Open Claude Code and paste this. Claude does the rest.

> Install ReStack: run **`git clone --single-branch --depth 1 https://github.com/zsh2401/ReStack.git ~/.claude/skills/restack && cd ~/.claude/skills/restack && ./setup ~/.claude/skills`** then add a "ReStack" section to CLAUDE.md that says available skills: /research, /latex-template, /docify, /peer-review. Then ask the user if they also want to add ReStack to the current project so teammates get it.

### Step 2: Add to your repo so teammates get it (optional)

> Add ReStack to this project: run **`cp -Rf ~/.claude/skills/restack .claude/skills/restack && rm -rf .claude/skills/restack/.git && cd .claude/skills/restack && ./setup .claude/skills`** then add a "ReStack" section to this project's CLAUDE.md that says available skills: /research, /latex-template, /docify, /peer-review, and tells Claude that if ReStack skills aren't working, run `cd .claude/skills/restack && ./setup .claude/skills` to reinstall.

Real files get committed to your repo (not a submodule), so `git clone` just works. Everything lives inside `.claude/`. Nothing touches your PATH or runs in the background.

> **Contributing or need full history?** The commands above use `--depth 1` for a fast install. If you plan to contribute or need full git history, do a full clone instead:
> ```bash
> git clone https://github.com/zsh2401/ReStack.git ~/.claude/skills/restack
> ```

## See it work

```
You:    /research Transformer attention mechanism survey 2024
Claude: [分类: 文献类 → OpenAlex → Semantic Scholar → Tavily]
        [Query rewrite: "Transformer attention mechanisms" + "attention survey 2024"]
        [OpenAlex: 2,341 结果, Semantic Scholar: 引用图展开, Tavily: 最新博客]
        [去重: 合并 DOI, rerank by citation count + recency]

        | # | Title | Authors | Year | Source | Citations |
        |---|-------|---------|------|--------|-----------|
        | 1 | Attention Is All You Need | Vaswani et al. | 2017 | openalex | 120,000+ |
        | 2 | A Survey on Attention Mechanisms... | ... | 2024 | openalex | 234 |
        | 3 | FlashAttention: Fast and Memory-Efficient... | ... | 2022 | semantic_scholar | 5,600+ |

        该领域2024年出现3篇主要综述，代表性方向包括稀疏注意力、线性注意力和硬件感知注意力。
        推荐下一步: 下载 FlashAttention 和最新综述全文精读。

You:    /latex-template NeurIPS 2026 LaTeX template
Claude: [分类: 具体会议模板 → Tavily → Brave]
        [Tavily: 搜索 "NeurIPS 2026 official LaTeX template author guidelines"]
        [Brave: 补充 GitHub / CTAN / Overleaf]
        [找到官方 .sty 文件 + Overleaf 镜像]

        ✅ NeurIPS 2026 Official LaTeX Template
        - 来源: neurips.cc (官方)
        - 下载: https://media.neurips.cc/.../neurips_2026.zip
        - Overleaf: https://www.overleaf.com/latex/templates/neurips-2026/...
        - 编译: pdflatex
        - 注意: 截止日期 2026-06-05, 需用 \texttt{neurips_2026.sty}

You:    /docify
Claude: [总结本次会话: Transformer 注意力机制综述 + NeurIPS 模板确认]
        → 保存到 docs/2026-06-02-transformer-attention-survey/index.md
        → 含引用清单 references.md
```

## The skills

ReStack 专注于科研工作的完整链路：搜索 → 模板 → 写作 → 归档。

| Skill | Your specialist | What they do |
|-------|----------------|--------------|
| `/research` | **Literature Search** | 多源学术搜索。先分类再检索：文献类走 OpenAlex → Semantic Scholar → Tavily → Brave；技术文档类走 Tavily → Brave。Query rewrite、去重、rerank、来源校验。六份数据源参考文档。 |
| `/latex-template` | **Template Finder** | LaTeX/Overleaf 模板检索。优先搜索官方源（出版社/CTAN/GitHub），避免过期第三方模板。支持期刊、会议、学校论文模板。 |
| `/docify` | **Research Archivist** | 会话归档。将当前研究成果保存到 `docs/YYYY-MM-DD-<title>/index.md`，支持附件和引用清单。 |
| `/peer-review` | **Peer Reviewer** | 结构化同行评审。方法学评估、统计有效性、报告标准合规（CONSORT/STROBE），带 checklist 的正式审稿。 |

## Design

ReStack is minimal by design:

- **Each skill is a directory with a SKILL.md.** No build step. No binary. Just Markdown.
- **References live alongside the skill.** `research/references/openalex.md` — the skill reads it at invocation time.
- **API keys are managed simply.** `~/.restack/<name>_key` — if the file exists, use the key. If it says `no`, skip. If it doesn't exist, ask once and remember.
- **setup is a single shell script.** It copies skills, injects update-check metadata into frontmatter, and installs to any target directory.

## 数据源覆盖

research 技能内置六份数据源参考（`references/`），每份覆盖完整 API：

| 数据源 | 用途 | 需要 Key |
|--------|------|:--------:|
| OpenAlex | 文献主索引（2.5 亿+ 文献） | 可选 |
| Semantic Scholar | 引用关系、相关推荐、影响力评分 | 可选 |
| Crossref | DOI 查询、出版元数据、期刊信息 | 否 |
| arXiv | AI/CV/NLP/物理 最新预印本 | 否 |
| Tavily | AI 优化网页搜索 + 正文提取 | 必须 |
| Brave | 独立搜索引擎 + LLM 片段 | 必须 |

## Troubleshooting

**Skill not showing up?** `cd ~/.claude/skills/restack && ./setup ~/.claude/skills`

**Stale install?** `cd ~/.claude/skills/restack && git pull && ./setup ~/.claude/skills`

**Claude says it can't see the skills?** Make sure your project's `CLAUDE.md` has a ReStack section. Add this:

```
## ReStack
Available skills: /research, /latex-template, /docify, /peer-review.
If ReStack skills aren't working, run:
cd .claude/skills/restack && ./setup .claude/skills
```

## License

MIT. Free forever. Go build something.
