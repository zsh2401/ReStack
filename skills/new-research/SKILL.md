---
name: new-research
description: 初始化研究项目 — 询问研究方向、写入 CLAUDE.md/AGENTS.md、建立 RESEARCH.md 索引
---

# 初始化研究项目

当用户说"开始一个新研究"、"init research"、"new-research"、或在空项目中表达研究意图时，按此流程初始化。

## 工作流程

### 第一步：收集信息

使用 AskUserQuestion 询问用户（一次 2-3 问）：

**问题 1: 研究主题/标题**

让用户自由输入简短描述。

**问题 2: 研究类型**

- 文献综述 (literature review)
- 实验研究 (experimental)
- 理论分析 (theoretical)
- 实证调查 (empirical)
- 系统构建 (system building)
- 其他（用户自定义）

**问题 3: 需要的 ReStack 技能**

多选，让用户选择需要激活的技能：

- `/research` — 学术文献搜索
- `/latex-template` — LaTeX 模板检索
- `/new-paper` — 新建论文
- `/docify` — 会话归档

### 第二步：检测并更新 CLAUDE.md / AGENTS.md

检查当前目录是否存在以下文件（按优先级）：

1. `CLAUDE.md`
2. `AGENTS.md`
3. `.claude/CLAUDE.md`
4. `.claude/AGENTS.md`

#### 如果文件已存在

**不要覆盖原有内容。** 读取现有文件，在末尾追加 ReStack 研究区块：

```markdown

---

## 研究项目

本目录是一个研究项目。

研究主题: <用户输入的主题>
研究类型: <用户选择的类型>
初始化日期: YYYY-MM-DD

### ReStack 研究技能

<根据用户选择列出可用技能>

研究工作流程:
1. 使用 `/research` 进行学术文献搜索
2. 使用 `/new-paper` 创建论文草稿
3. 使用 `/docify` 归档研究成果到 docs/

### 研究索引

研究成果索引: [RESEARCH.md](RESEARCH.md)
```

#### 如果文件不存在

创建 `CLAUDE.md`（AGENTS.md 如用户偏好），内容包含上述区块 + 基本的 Claude Code 研究指令：

```markdown
# <项目名称或目录名>

## 研究项目

本目录是一个研究项目。

研究主题: <主题>
研究类型: <类型>
初始化日期: YYYY-MM-DD

## ReStack 研究技能

<根据用户选择列出可用技能>

可用技能: <技能列表>

研究工作流程:
1. 使用 /research 进行学术文献搜索
2. 使用 /new-paper 创建论文草稿
3. 使用 /docify 归档研究成果到 docs/

## 研究索引

研究成果索引: [RESEARCH.md](RESEARCH.md)

## 指令

- 本目录用于学术研究，优先使用研究工具
- 输出保持学术严谨性，带引用和来源标注
- 不确定时，优先搜索文献而非推测
```

### 第三步：创建 RESEARCH.md

`RESEARCH.md` 是一份**动态研究文档**，随研究推进持续更新。它记录研究目标、论文结构、工作进度等。

在项目根目录创建（如果已存在则更新 `## 研究目标` 部分）：

```markdown
# <研究主题>

> 初始化日期: YYYY-MM-DD | 类型: <研究类型> | 状态: active

## 研究目标

- [ ] TODO: <用户输入的目标，或占位>
- [ ] TODO: <子目标 2>
- [ ] TODO: <子目标 3>

## 论文结构

| 章节 | 内容 | 文件 | 状态 |
|------|------|------|------|
| Abstract | 摘要 | — | pending |
| Introduction | 背景与问题定义 | — | pending |
| Related Work | 相关工作综述 | — | pending |
| Method | 方法描述 | — | pending |
| Experiments | 实验设计与结果 | — | pending |
| Conclusion | 结论与展望 | — | pending |
| References | 引用清单 | — | pending |

> 使用 /new-paper 创建论文后，此表自动关联实际文件路径（见 papers/ 目录）。
> 使用 /research 填充 Related Work 的文献基础。

## 工作进度

| 日期 | 进度 | 产出 | 备注 |
|------|------|------|------|
| YYYY-MM-DD | 初始化 | RESEARCH.md CLAUDE.md | 研究启动 |

## 文献清单

| # | 标题 | 作者 | 年 | 来源 | 关系 | 链接 |
|---|------|------|----|------|------|------|
| — | — | — | — | — | — | — |

> 使用 /research 搜索文献后追加到此表。

## 论文输出

| 目录 | 标题 | 模板 | 目标 | 状态 |
|------|------|------|------|------|
| — | — | — | — | — |

> 使用 /new-paper 创建论文后自动填充。
```

### RESEARCH.md 更新规则

后续使用 ReStack 技能时，按以下规则更新 RESEARCH.md：

- **/research** 完成文献搜索 → 追加 `## 文献清单` 中的条目
- **/new-paper** 创建论文 → 更新 `## 论文结构` 中的文件路径，追加 `## 论文输出` 行
- **/docify** 归档 → 追加 `## 工作进度` 行，记录产出
- **研究目标变更** → 更新 `## 研究目标` 中的 checklist
- **手动编辑** → 用户可随时直接编辑，技能会读取当前状态后追加

### 第四步：完成后输出

```
✅ 研究项目已初始化

目录: <当前目录>
主题: <主题>
类型: <类型>
CLAUDE.md: <已创建 / 已更新>
RESEARCH.md: <已创建 / 已更新>
可用技能: <用户选择的技能列表>

下一步:
- /research 搜索相关文献，填充学术背景
- /new-paper 创建论文草稿
- 使用 latex-template 技能查找投稿模板
```

### 注意事项

- 不要覆盖用户已有的 CLAUDE.md 内容，只能追加 ReStack 区块
- 如果用户已有 CLAUDE.md，且已包含研究相关内容，只更新 RESEARCH.md 索引
- 如果目录已经是 git 仓库，提醒用户 RESEARCH.md 可以版本追踪
