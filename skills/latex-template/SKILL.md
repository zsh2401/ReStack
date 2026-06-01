---
name: latex-template
description: LaTeX / Overleaf 模板检索与下载 — 根据期刊、会议、学校或模板名称找到可靠模板源
---

# LaTeX 模板检索

根据用户给出的期刊、会议、学校、论文类型、模板名称或 Overleaf 链接，找到可靠的 LaTeX 模板来源，优先返回可下载、可导入 Overleaf、可本地编译的模板包。

## 搜索策略

### 问题分流

**先判断 query 是什么类型，不要错用此技能：**

- 如果是搜索**论文文献**（找论文、读论文、文献综述等）→ **不要用此技能**，转到 OpenAlex / Semantic Scholar 进行文献检索
- 如果是搜索**模板或文档**（LaTeX 模板、投稿指南、样式文件、文档类等）→ 继续本技能

### 通用模板/文档搜索

```
Tavily → Brave
```

### 具体期刊/会议论文模板搜索

如果 query 是确定名称的期刊或会议（如 "IEEE Access template"、"NeurIPS LaTeX template"）：

```
1. Tavily: 搜索 "official LaTeX template" / "author guidelines" / "submission template"
2. Brave: 补充搜索 GitHub / CTAN / Overleaf Gallery 镜像
```

## 检索原则

### 搜索优先级

不要默认只搜 Overleaf Gallery。目标源的搜索顺序为：

1. **期刊/会议/出版社/学校官网**（最优先，官方源）
2. **CTAN**（https://ctan.org）
3. **GitHub 官方仓库**（release zip 或 main branch zip）
4. **Overleaf Gallery**（https://www.overleaf.com/gallery 或 /latex/templates）
5. **普通网页搜索 fallback**

### 按输入类型分流

#### 期刊或会议名称

用户给出如 "IEEE Access"、"ACM MM"、"NeurIPS"、"ICCV"、"Applied Intelligence" 时，搜索：

- `"<名称> official LaTeX template"` — 找官方模板
- `"<名称> author guidelines"` — 找投稿指南中的模板链接
- `"<名称> submission template"` — 找投稿模板
- `"<名称> Overleaf template"` — 找 Overleaf 镜像
- `"<名称> CTAN"` — 找 CTAN 包

#### 学校论文模板

用户给出学校名称（如 "MIT thesis template"、"浙江大学毕业论文模板"）时，搜索：

- 学校研究生院 / 图书馆 / 学院官网的 "thesis template" / "dissertation template"
- GitHub 上 `<学校> thesis latex template`
- Overleaf Gallery 上的学校 thesis 模板

#### Overleaf Gallery 链接

用户给出 Overleaf Gallery 页面时：

1. 识别模板标题、作者、是否官方维护、最后更新时间
2. 检查页面是否提供 GitHub / CTAN / Publisher 外部链接 → 优先使用外部源
3. 如需下载源码：说明需在 Overleaf 中 "Open as Template"，然后从项目菜单 Download → Source (.zip)
4. 提醒：Overleaf 上的模板可能被第三方上传，非官方

### 找到 GitHub 仓库时

- 返回仓库 URL
- 提供 release zip 链接（`/releases/latest`）或 main branch zip（`/archive/refs/heads/main.zip`）
- 说明可 `git clone` 到本地，或 .zip 上传到 Overleaf

### 找到 CTAN 包时

- 返回 CTAN 页面 URL
- 返回包名（如 `ieeetran`）
- 返回文档 PDF 链接
- 返回源码包下载链接

### 投稿检查

如果模板用于投稿，优先检查是否来自官方 publisher/conference 页面。避免使用：
- 过期的第三方模板（标题页可能不符合最新要求）
- 个人修改版（可能缺少必需字段如 ORCID、CCS 概念、版权声明等）

### 禁止行为

- 禁止编造模板下载链接、版本号、作者、更新时间
- 禁止声称某个模板是"官方"除非有明确来源证明
- 如果没有找到官方模板，必须明确说明，并给出最接近的替代模板 + 风险提示

## 输出格式

```json
{
  "query": "用户原始输入",
  "template_name": "模板名称",
  "official": true,
  "source_type": "publisher | ctan | github | overleaf_gallery | third_party",
  "download": {
    "type": "direct_zip | github_release | git_clone | ctan_package | overleaf_import",
    "url": "下载链接（不要编造）"
  },
  "overleaf_compatible": true,
  "last_updated": "2024-06-01（如有）",
  "notes": "使用注意事项（如投稿截止日期相关、编译器要求 LuaLaTeX 等）",
  "alternatives": [
    {
      "name": "替代模板名",
      "url": "...",
      "risk": "第三方维护，可能过期"
    }
  ]
}
```

## 典型回复格式

找到官方模板时：

> **<模板名称>** — 官方 <期刊/会议/学校> LaTeX 模板
>
> - 来源：<publisher 官网>
> - 下载：<直接链接>
> - Overleaf：<Overleaf 链接（如有）>
> - 编译：<pdflatex / xelatex / lualatex>
> - 注意：<关键注意事项>

未找到时：

> 未找到 <名称> 的官方 LaTeX 模板。以下是替代方案：
>
> 1. **<替代名>**（第三方，<风险说明>）— <链接>
> 2. **通用模板** — 可用 <CTAN 包名> 自行配置
>
> 建议联系期刊编辑部确认最终模板要求。
