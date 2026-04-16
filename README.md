# stata-graphics-skill

A structured knowledge base for [Claude Code](https://claude.ai/code) that guides the AI to produce publication-quality Stata graphics — especially for empirical economics research.

为 [Claude Code](https://claude.ai/code) 构建的 Stata 作图知识库，帮助 AI 生成高质量的 Stata 图形，尤其适合经济学实证研究场景。

---

## English

### What Is This?

When you ask Claude Code to write Stata graphics code, it relies on its training data — which can be inconsistent for complex tasks like event study plots, coefficient comparison figures, or multi-panel heterogeneity analyses. This skill changes that behavior.

**Before this skill:** CC writes code from memory → user spots errors → multiple rounds of correction.

**After this skill:** CC consults reference files first → writes code → user makes minor adjustments.

This is a *preventive* supervision mechanism, not a post-hoc fix.

### What's Covered

**Estimation result visualization** (highest priority — the hardest part for LLMs):

| Figure type | Technical components |
|-------------|---------------------|
| Event study / parallel trends | `rcap` + `scatter` overlay, custom x-axis labels, reference lines |
| Coefficient plots (single / multi-model) | `coefplot` or manual `rcap` + `scatter`, horizontal/vertical |
| Heterogeneity analysis (multi-panel) | Horizontal coefficient plots, group labels, `graph combine` |
| Marginal effects | `margins` + `marginsplot` or manual extraction |
| Permutation / placebo test | Dual y-axis, kernel density + p-value scatter overlay |
| DiD dynamic effects | `csdid_plot` or manual event plot |

**Descriptive and data visualization** (supplementary):
trend lines, distributions, scatter with fit, bar charts, box plots.

### Three-Layer Architecture

```
Layer 1 — existing stata skill (dylantmoore/stata-skill)
  Knows how to run regressions; knows e() structures for reghdfe, csdid, etc.

Layer 2 — estimation-to-graph.md  [bridge, built first]
  Knows how to extract stored results and reshape them into plottable data

Layer 2b — graph-templates.md  [built alongside the bridge]
  Ready-to-run code templates for common figure types (event study, coefplot,
  csdid_plot, margins); grows over time via /learn from real-world examples

Layer 3 — stata-graphics skill  [this repo]
  Knows graph style details: markers, colors, axes, schemes, CI methods, export
```

Each layer is necessary. Missing any one creates gaps in the workflow.

### Prerequisites

- [Claude Code](https://claude.ai/code) installed
- [dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill) installed at `~/.claude/stata/`
  (macOS/Linux) or `%USERPROFILE%\.claude\stata\` (Windows)
- Stata 14 or later (core syntax covered is stable across versions)

### Installation

1. Clone or download this repository.

2. Create the skill directory in your Claude config folder:
   - macOS/Linux: `mkdir -p ~/.claude/stata-graphics/references`
   - Windows: `mkdir %USERPROFILE%\.claude\stata-graphics\references`

3. Copy completed reference files:
   ```
   # macOS/Linux
   cp references/*.md ~/.claude/stata-graphics/references/

   # Windows (PowerShell)
   Copy-Item references\*.md $env:USERPROFILE\.claude\stata-graphics\references\
   ```

4. When `SKILL.md` is ready (Session G), copy it to the root of `stata-graphics/`:
   ```
   cp SKILL.md ~/.claude/stata-graphics/SKILL.md
   ```

5. Add the trigger rule to your global `~/.claude/CLAUDE.md`:
   ```
   ## Stata Graphics Skill Reference
   When writing Stata graphics code—or when the task involves producing a figure,
   plot, or visual output from Stata estimation results—read
   ~/.claude/stata-graphics/SKILL.md for graphics syntax, options reference,
   and routing to relevant files. This applies to both explicit graph requests
   and implicit ones (e.g., "show the event study coefficients", "plot parallel
   trends", "visualize the results").
   ```

### Build Status

| File | Status | Description |
|------|--------|-------------|
| `estimation-to-graph.md` | ✅ | Bridge: regression results → plottable data |
| `graph-templates.md` | ✅ | Ready-to-run templates (event study, coefplot, margins, etc.); grows via `/learn` |
| `ci-bands.md` | ✅ | rcap / rspike / rarea comparison + event study template |
| `twoway-syntax.md` | ✅ | All twoway plottypes (43), overlay syntax, twoway_options, addplot(), recast() |
| `axes.md` | ✅ | xlabel/ylabel, xscale/yscale, reference lines, dual y-axis |
| `schemes-regions.md` | ✅ | Scheme comparison, graph regions, graph size, publication style |
| `markers.md` | ✅ | msymbol (all symbolstyles), msize, mcolor/mfcolor/mlcolor, markerstyle presets |
| `lines.md` | ✅ | lpattern (named + formula), lwidth, lcolor, lstyle, connect_options |
| `colors.md` | ✅ | Named colors, RGB/CMYK/HSV/hex, opacity (%), intensity (*), palettes package |
| `labels-text.md` | ✅ | title/subtitle/note/caption, text()/ttext(), textbox_options, mlabel/mlab* |
| `legend.md` | ⬜ | legend() complete options |
| `export-combine.md` | ⬜ | graph export formats, graph combine layouts |
| `by-over.md` | ⬜ | by() vs over() vs graph combine |
| `SKILL.md` | ⬜ | Routing table (written last) |

Status: ⬜ Not started · 🔄 In progress · ✅ Complete

### Reference Sources

- **Mitchell (2022)** — *A Visual Guide to Stata Graphics*, 4th ed. (Stata Press)
- **Stata Graphics Reference Manual `[G]`** — bundled with Stata; access via `help graph`
- **[dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)** — source for the bridge file

### Contributing

This skill improves through use. If you find an error or add a new chart type:
1. Fix or extend the relevant reference file
2. To add a new figure template from a real-world example, use the `/learn` skill and direct it to append to `graph-templates.md`
3. Note the source and Stata version
4. Open a PR or issue

---

## 中文

### 这是什么？

当你让 Claude Code 写 Stata 画图代码时，它依赖训练数据——对于复杂任务（如 event study 图、系数对比图、异质性分析图），这种依赖往往导致反复出错、多轮纠正。这个 skill 改变了这一行为模式。

**没有 skill 时：** CC 凭记忆写代码 → 用户发现错误 → 多轮纠正。

**有 skill 后：** CC 先查阅参考文件 → 写代码 → 用户做少量微调。

这是一种**预防性监督机制**，不是事后补救。

### 覆盖范围

**估计结果可视化**（最高优先级，也是 LLM 最容易出错的场景）：

| 图类型 | 技术构成 |
|--------|---------|
| Event study / 平行趋势检验图 | `rcap` + `scatter` 叠加，自定义 x 轴标签，参考线 |
| 系数图（单模型 / 多模型对比） | `coefplot` 或手工 `rcap` + `scatter` |
| 异质性分析图（多 panel） | 横向系数图，分组标签，`graph combine` 拼合 |
| 边际效应图 | `margins` + `marginsplot` 或手工提取后画图 |
| Permutation / placebo test 图 | 双 y 轴，kernel density + p-value scatter 叠加 |
| DiD 动态效应图 | `csdid_plot` 或手工 event plot |

**描述性与数据可视化**（按需补充）：趋势折线图、分布图、散点图、分组柱状图、箱线图。

### 三层架构

```
第一层 — 现有 stata skill（dylantmoore/stata-skill）
  知道怎么跑回归，知道 reghdfe、csdid 等命令的 e() 结构

第二层 — estimation-to-graph.md（桥梁文件，最先建设）
  知道如何把存储结果转化为可画图的数据格式

第二层b — graph-templates.md（与桥梁文件同期建设）
  各类图形的完整即用代码模板（event study、coefplot、
  csdid_plot、margins 等）；通过 /learn 从实际论文中持续扩充

第三层 — stata-graphics skill（本项目）
  知道图的样式细节：marker、颜色、坐标轴、scheme、CI 画法、导出
```

三层缺任何一层都有短板。

### 前置条件

- 已安装 [Claude Code](https://claude.ai/code)
- 已安装 [dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)，位于：
  - macOS/Linux：`~/.claude/stata/`
  - Windows：`%USERPROFILE%\.claude\stata\`
- Stata 14 及以上版本（本 skill 覆盖的核心语法跨版本稳定）

### 安装方法

1. 克隆或下载本仓库。

2. 在 Claude 配置目录下创建 skill 文件夹：
   - macOS/Linux：`mkdir -p ~/.claude/stata-graphics/references`
   - Windows：`mkdir %USERPROFILE%\.claude\stata-graphics\references`

3. 复制参考文件：
   ```bash
   # macOS/Linux
   cp references/*.md ~/.claude/stata-graphics/references/

   # Windows (PowerShell)
   Copy-Item references\*.md $env:USERPROFILE\.claude\stata-graphics\references\
   ```

4. `SKILL.md` 完成后（Session G），复制到 `stata-graphics/` 根目录：
   ```bash
   cp SKILL.md ~/.claude/stata-graphics/SKILL.md
   ```

5. 在全局 `~/.claude/CLAUDE.md`（Windows：`%USERPROFILE%\.claude\CLAUDE.md`）中添加触发规则：
   ```
   ## Stata Graphics Skill Reference
   When writing Stata graphics code—or when the task involves producing a figure,
   plot, or visual output from Stata estimation results—read
   ~/.claude/stata-graphics/SKILL.md for graphics syntax, options reference,
   and routing to relevant files. This applies to both explicit graph requests
   and implicit ones (e.g., "show the event study coefficients", "plot parallel
   trends", "visualize the results").
   ```

### 构建进度

| 文件 | 状态 | 说明 |
|------|------|------|
| `estimation-to-graph.md` | ✅ | 桥梁：回归结果 → 可画图数据 |
| `graph-templates.md` | ✅ | 即用代码模板（event study、coefplot、margins 等）；通过 `/learn` 持续扩充 |
| `ci-bands.md` | ✅ | rcap/rspike/rarea 对比 + event study 模板 |
| `twoway-syntax.md` | ✅ | 全部 43 种 plottype、overlay 语法、twoway_options、addplot()、recast() |
| `axes.md` | ✅ | xlabel/ylabel、xscale/yscale、参考线、双 y 轴 |
| `schemes-regions.md` | ✅ | scheme 对比、图形区域、图形尺寸、发表级规范 |
| `markers.md` | ✅ | msymbol（完整 symbolstyle 表）、msize、mcolor/mfcolor/mlcolor、markerstyle 预设 |
| `lines.md` | ✅ | lpattern（命名 + 公式语法）、lwidth、lcolor、lstyle、connect_options |
| `colors.md` | ✅ | 命名色、RGB/CMYK/HSV/hex、opacity (%)、intensity (*)、palettes 包 |
| `labels-text.md` | ✅ | title/subtitle/note/caption、text()/ttext()、textbox_options、mlabel/mlab* |
| `legend.md` | ⬜ | legend() 完整选项 |
| `export-combine.md` | ⬜ | graph export 格式、graph combine 布局 |
| `by-over.md` | ⬜ | by() vs over() vs graph combine |
| `SKILL.md` | ⬜ | 路由表（最后写） |

状态：⬜ 未开始 · 🔄 进行中 · ✅ 已完成

### 参考资料

- **Mitchell (2022)** — *A Visual Guide to Stata Graphics*，第四版（Stata Press）
- **Stata Graphics Reference Manual `[G]`** — 随 Stata 安装附带，在 Stata 中输入 `help graph` 访问
- **[dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)** — 桥梁文件的来源

### 贡献

这个 skill 在使用中不断完善。如果你发现错误，或者新增了图类型：
1. 修复或扩充对应的参考文件
2. 如需从实际论文/复现包中添加新模板，使用 `/learn` skill 并指定写入 `graph-templates.md`
3. 注明来源和 Stata 版本
4. 提交 PR 或 Issue
