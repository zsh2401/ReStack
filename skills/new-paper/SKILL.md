---
name: new-paper
description: 创建新论文项目 — 选择模板、初始化目录结构、写入 RESEARCH.md 追踪
---

# 新建论文

当用户说"写论文"、"new paper"、"开始写"等时，按以下流程初始化一个论文项目。

## 工作流程

### 第一步：收集信息

使用 AskUserQuestion 询问用户：

**问题 1: 论文标题/主题**

让用户自由输入。

**问题 2: 模板来源**

- 选项 A: **搜索模板** — 按期刊/会议/学校名称搜索，调用 latex-template 技能的检索逻辑（Tavily → Brave，找官方模板）
- 选项 B: **默认模板** — 使用内置的干净通用学术模板（适合 arxiv 预印本风格）

### 第二步：生成目录结构

在项目根目录创建：

```
papers/<YYYY-MM-DD-slug>/
├── main.tex
├── sections/
│   ├── abstract.tex
│   ├── introduction.tex
│   ├── related-work.tex
│   ├── method.tex
│   ├── experiments.tex
│   └── conclusion.tex
├── figures/
├── tables/
├── refs.bib
└── README.md
```

### 第三步：写入 main.tex

#### 如果用户选择"搜索模板"

使用 latex-template 技能的检索逻辑：
1. Tavily → 搜索 "`<名称>` official LaTeX template" / "author guidelines"
2. Brave → 补充 GitHub / CTAN / Overleaf Gallery
3. 找到后下载 .zip 或 .sty 文件
4. 按模板要求写入 main.tex

检查优先级：出版社官网 > CTAN > GitHub > Overleaf Gallery。
禁止编造下载链接；没有官方模板时明确告知并提供替代。

#### 如果用户选择"默认模板"

生成一个干净的通用模板，适合 arxiv/预印本风格：

```latex
\documentclass[11pt,a4paper]{article}

\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage{amsmath,amssymb,amsfonts}
\usepackage{graphicx}
\usepackage{hyperref}
\usepackage{natbib}
\usepackage[margin=1in]{geometry}
\usepackage{booktabs}
\usepackage{algorithm}
\usepackage{algpseudocode}
\usepackage{xcolor}

\hypersetup{
    colorlinks=true,
    linkcolor=blue,
    citecolor=blue,
    urlcolor=blue
}

\title{<PAPER TITLE>}
\author{<AUTHOR>}
\date{\today}

\begin{document}

\maketitle

\begin{abstract}
\input{sections/abstract}
\end{abstract}

\input{sections/introduction}
\input{sections/related-work}
\input{sections/method}
\input{sections/experiments}
\input{sections/conclusion}

\bibliographystyle{plainnat}
\bibliography{refs}

\end{document}
```

### 第四步：写入 sections 骨架

每个 section 文件写入占位内容：

```latex
% sections/abstract.tex
TODO: 摘要
```

```latex
% sections/introduction.tex
\section{Introduction}
\label{sec:introduction}

TODO: 背景、问题定义、贡献。
```

其他 section 同理，用 `\section{}` + `\label{}` + TODO 占位。

### 第五步：写入 refs.bib

创建空的 BibTeX 文件：

```bibtex
% References — use /research to find and add entries
```

### 第六步：写入 papers/<slug>/README.md

记录论文元数据：

```markdown
# <PAPER TITLE>

- 创建日期: YYYY-MM-DD
- 模板: <默认 | 期刊/会议名称>
- 目标: <投稿目标，如有>
- 状态: drafting
```

### 第七步：更新 RESEARCH.md

读取项目根目录 `RESEARCH.md`（若不存在则创建基本模板），更新以下三个部分：

#### 更新 `## 论文结构`

将每个 section 文件关联到章节行：

```markdown
## 论文结构

| 章节 | 内容 | 文件 | 状态 |
|------|------|------|------|
| Abstract | 摘要 | `papers/<slug>/sections/abstract.tex` | drafting |
| Introduction | 背景与问题定义 | `papers/<slug>/sections/introduction.tex` | drafting |
| Related Work | 相关工作综述 | `papers/<slug>/sections/related-work.tex` | drafting |
| Method | 方法描述 | `papers/<slug>/sections/method.tex` | drafting |
| Experiments | 实验设计与结果 | `papers/<slug>/sections/experiments.tex` | drafting |
| Conclusion | 结论与展望 | `papers/<slug>/sections/conclusion.tex` | drafting |
| References | 引用清单 | `papers/<slug>/refs.bib` | drafting |
```

章节状态选项: `pending`, `drafting`, `written`, `revised`, `final`。

#### 更新 `## 论文输出`

追加或更新条目：

```markdown
| papers/<slug>/ | <PAPER TITLE> | <模板> | <目标> | drafting |
```

#### 追加 `## 工作进度`

```markdown
| YYYY-MM-DD | 论文初始化 | main.tex, sections/, refs.bib | 使用 <模板> 模板 |
```

### 完成后

输出创建的目录树、下一步建议：

```
✅ 论文项目已创建: papers/2026-06-02-attention-survey/

目录结构:
├── main.tex
├── sections/ (6 files)
├── figures/
├── tables/
├── refs.bib
└── README.md

下一步:
- 用 /research 搜索相关文献，填充 refs.bib
- 编辑 sections/ 下的各章节
- 编译: cd papers/<slug> && pdflatex main.tex
- 完成后 /docify 归档研究上下文
```
