# stata-graphics-skill

为 AI 编程助手（Claude Code、Codex 等）构建的 Stata 作图知识库，帮助 AI 生成高质量的 Stata 图形，尤其适合经济学实证研究场景。

A structured knowledge base for AI coding assistants (Claude Code, Codex, etc.) that guides the AI to produce publication-quality Stata graphics — especially for empirical economics research.

---

## 中文

### 这是什么？

当你让 AI 编程助手写 Stata 画图代码时，它依赖训练数据——对于复杂任务（如 event study 图、系数对比图、异质性分析图），这种依赖往往导致反复出错、多轮纠正。这个 skill 改变了这一行为模式。

**没有 skill 时：** AI 凭记忆写代码 → 用户发现错误 → 多轮纠正。

**有 skill 后：** AI 先查阅参考文件 → 写代码 → 用户做少量微调。

这是一种**预防性监督机制**，不是事后补救。

### 覆盖范围

**估计结果可视化**（最高优先级，也是 AI 最容易出错的场景）：

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

第二层 — estimation-to-graph.md（桥梁文件）
  知道如何把存储结果转化为可画图的数据格式

第二层b — graph-templates.md（即用代码模板）
  各类图形的完整即用代码模板（event study、coefplot、
  csdid_plot、margins 等）；通过 /learn 从实际论文中持续扩充

第三层 — stata-graphics skill（本项目）
  知道图的样式细节：marker、颜色、坐标轴、scheme、CI 画法、导出
```

三层缺任何一层都有短板。

### 前置条件

**1. 安装 [dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)**

```bash
# 克隆仓库
git clone https://github.com/dylantmoore/stata-skill.git

# 创建目录并复制文件（macOS/Linux）
mkdir -p ~/.claude/stata
cp stata-skill/SKILL.md ~/.claude/stata/
cp -r stata-skill/references/ ~/.claude/stata/references/
cp -r stata-skill/packages/  ~/.claude/stata/packages/
```

在 `~/.claude/CLAUDE.md` 中添加触发规则：

```
## Stata Skill Reference
- When writing or debugging Stata code, read ~/.claude/stata/SKILL.md
  for syntax reference and package documentation
```

**2. Stata 14 及以上版本**（本 skill 覆盖的核心语法跨版本稳定）

### 安装本 skill（Claude Code）

**1. 克隆本仓库：**

```bash
git clone https://github.com/YoungFujun/stata-graphics-skill.git
```

**2. 创建目录并复制文件：**

```bash
# macOS/Linux
mkdir -p ~/.claude/stata-graphics/references
cp stata-graphics-skill/SKILL.md ~/.claude/stata-graphics/
cp stata-graphics-skill/references/*.md ~/.claude/stata-graphics/references/
```

**3. 在 `~/.claude/CLAUDE.md` 中添加触发规则：**

```
## Stata Graphics Skill Reference
- When writing Stata graphics code—or when the task involves producing a
  figure, plot, or visual output from Stata estimation results—read
  ~/.claude/stata-graphics/SKILL.md for graphics syntax, options reference,
  and routing to relevant files. This applies to both explicit graph requests
  and implicit ones (e.g., "show the event study coefficients", "plot parallel
  trends", "visualize the results").
```

### 在 Codex CLI 上使用本 skill

本 skill 的参考文件是标准 Markdown，可以移植到任何能读取本地文件的 AI 编程助手。以 OpenAI Codex CLI 为例：

**1. 复制文件到 Codex 配置目录：**

```bash
# macOS/Linux
mkdir -p ~/.codex/stata-graphics/references
cp stata-graphics-skill/SKILL.md ~/.codex/stata-graphics/
cp stata-graphics-skill/references/*.md ~/.codex/stata-graphics/references/
```

**2. 在 `~/.codex/instructions.md` 中添加触发规则：**

```
## Stata Graphics Skill Reference
When writing Stata graphics code, or when the task involves producing a figure
from Stata estimation results, read ~/.codex/stata-graphics/SKILL.md first for
graphics syntax, options, and routing to the relevant reference files.
```

**适配其他助手：** 将文件路径换成对应助手的配置目录（如 `~/.gemini/`、`.cursor/` 等），在全局指令文件中添加相同触发规则即可。路由逻辑和参考文件本身不依赖任何特定助手。

### 从实际论文学习模板

本 skill 的一项核心能力是从真实论文图形中提炼可复用的代码模板，持续扩充 `graph-templates.md`。

#### 有源代码时

当你拿到一张好图和对应的复现代码，可以这样告诉 AI：

> "这是 [作者名] 论文中 [图类型] 的代码，帮我提炼其中的关键技法，写入 `graph-templates.md`。"

AI 会：
1. 识别代码中非显而易见的技法
2. 剥离论文特定的变量名，提炼为通用模板
3. 为每个非常规选项添加注释说明
4. 将模板追加到 `graph-templates.md`

**示例**：下图来自 Huang & Zhang (2021)，AEJ: Applied，展示了按性别、教育程度、收入水平分组的异质性回归系数（横向系数图，三个子面板，分组标签直接标注在系数点旁）。

> 将论文图片保存为 `assets/figure3_panel_b.png` 后可在此处显示：
> `![Huang & Zhang 2021, Figure 3 Panel B](assets/figure3_panel_b.png)`

通过分析此图的源代码，AI 提炼出如下通用技法并写入了 `graph-templates.md`（Section 6A）：

| 技法 | 说明 |
|------|------|
| `replace n = -n` | 对行号取反，使图形按列表顺序从上到下排列（不取反则从下到上） |
| 空行作为视觉分隔 | `coef`/`se` 缺失的行在图中产生空白间隔，同一行位置用于 `text()` 面板标题 |
| `xscale(alt)` | 将 x 轴从默认底部移至图的顶部；罕见选项，产生"轴在上方"的布局 |
| `yscale(off)` + `ylabel(... , noticks)` | 隐藏所有 y 轴数字标签，同时保留坐标结构供 `text()` 定位 |
| `mlabp(12)` | 将组标签置于标记点正上方（12 点钟方向）；`hori` 状态下显示在点的斜上方 |
| `text(y x "...", place(e))` | 面板标题锚定到空行的 y 坐标和 x 轴左端；`place(e)` 使文字向右展开 |

#### 仅有图片时

即使没有源代码，只要有 Stata 图片，AI 也可以借助本 skill 重建画图思路：

- 从图形外观识别图类型（横向系数图、event study、RD 等）
- 路由到相关参考文件（`estimation-to-graph.md`、`markers.md`、`axes.md` 等）
- 根据参考文件中的选项清单推导出可能的实现方案

**局限性说明**：仅凭图片，AI 无法推断以下非视觉信息：

- 数据构建方式（e.g., `replace n = -n` 这类索引技巧）
- 具体数值参数（xlabel 范围、text() 坐标）
- 罕见选项的存在（e.g., `xscale(alt)`）

因此，图片 + 源代码的组合可以提炼出精度更高的通用模板。

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
| `legend.md` | ✅ | legend() 的 contents/location、order/label/lastlabel、by() 特殊规则 |
| `export-combine.md` | ✅ | saving/use/display 工作流、graph combine 对齐、graph export 格式 |
| `graph-bar-box.md` | ✅ | 轻量版：经济学常见 bar/box/dot 图 |
| `by-over.md` | ✅ | F 的主文件：over() vs by() vs graph combine 的选择规则 |
| `SKILL.md` | ✅ | 路由表：任务类型 → 参考文件、10 条高频陷阱、stata skill 联动说明 |

状态：⬜ 未开始 · 🔄 进行中 · ✅ 已完成

### 参考资料

- **Mitchell (2022)** — *A Visual Guide to Stata Graphics*，第四版（Stata Press）
- **Stata Graphics Reference Manual `[G]`** — 随 Stata 安装附带，在 Stata 中输入 `help graph` 访问
- **[dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)** — 桥梁文件的来源

### 贡献

这个 skill 在使用中不断完善：
1. 修复或扩充对应的参考文件
2. 从实际论文中添加新模板：将图片（和源代码）提供给 AI，指定写入 `graph-templates.md`
3. 注明来源（作者、期刊、年份）和 Stata 版本
4. 提交 PR 或 Issue

---

## English

### What Is This?

When you ask an AI coding assistant to write Stata graphics code, it relies on training data — which can be inconsistent for complex tasks like event study plots, coefficient comparison figures, or multi-panel heterogeneity analyses. This skill changes that behavior.

**Without this skill:** AI writes code from memory → user spots errors → multiple rounds of correction.

**With this skill:** AI consults reference files first → writes code → user makes minor adjustments.

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

**Descriptive and data visualization** (supplementary): trend lines, distributions, scatter with fit, bar charts, box plots.

### Three-Layer Architecture

```
Layer 1 — existing stata skill (dylantmoore/stata-skill)
  Knows how to run regressions; knows e() structures for reghdfe, csdid, etc.

Layer 2 — estimation-to-graph.md  [bridge]
  Knows how to extract stored results and reshape them into plottable data

Layer 2b — graph-templates.md
  Ready-to-run code templates for common figure types; grows via /learn

Layer 3 — stata-graphics skill  [this repo]
  Knows graph style details: markers, colors, axes, schemes, CI methods, export
```

### Prerequisites

**1. Install [dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)**

```bash
git clone https://github.com/dylantmoore/stata-skill.git
mkdir -p ~/.claude/stata
cp stata-skill/SKILL.md ~/.claude/stata/
cp -r stata-skill/references/ ~/.claude/stata/references/
cp -r stata-skill/packages/  ~/.claude/stata/packages/
```

Add to `~/.claude/CLAUDE.md`:

```
## Stata Skill Reference
- When writing or debugging Stata code, read ~/.claude/stata/SKILL.md
  for syntax reference and package documentation
```

**2. Stata 14 or later** (core syntax covered is stable across versions)

### Installation (Claude Code)

**1. Clone this repository:**

```bash
git clone https://github.com/YoungFujun/stata-graphics-skill.git
```

**2. Copy files to Claude config directory:**

```bash
# macOS/Linux
mkdir -p ~/.claude/stata-graphics/references
cp stata-graphics-skill/SKILL.md ~/.claude/stata-graphics/
cp stata-graphics-skill/references/*.md ~/.claude/stata-graphics/references/
```

**3. Add trigger rule to `~/.claude/CLAUDE.md`:**

```
## Stata Graphics Skill Reference
- When writing Stata graphics code—or when the task involves producing a
  figure, plot, or visual output from Stata estimation results—read
  ~/.claude/stata-graphics/SKILL.md for graphics syntax, options reference,
  and routing to relevant files. This applies to both explicit graph requests
  and implicit ones (e.g., "show the event study coefficients", "plot parallel
  trends", "visualize the results").
```

### Using with Codex CLI

The reference files are plain Markdown and portable to any AI coding assistant that can read local files. For OpenAI Codex CLI:

**1. Copy files to Codex config directory:**

```bash
mkdir -p ~/.codex/stata-graphics/references
cp stata-graphics-skill/SKILL.md ~/.codex/stata-graphics/
cp stata-graphics-skill/references/*.md ~/.codex/stata-graphics/references/
```

**2. Add trigger rule to `~/.codex/instructions.md`:**

```
## Stata Graphics Skill Reference
When writing Stata graphics code, or when the task involves producing a figure
from Stata estimation results, read ~/.codex/stata-graphics/SKILL.md first for
graphics syntax, options, and routing to the relevant reference files.
```

**For other assistants:** Replace the config path with your assistant's equivalent directory (e.g., `~/.gemini/`, `.cursor/`). The routing logic and reference files themselves do not depend on any specific assistant.

### Learning from Real-World Examples

A core capability of this skill is extracting reusable code templates from real published figures. These templates are appended to `graph-templates.md` and become immediately available for future tasks.

#### With source code

When you have both a figure and its replication code, tell the AI:

> "This is the code for [figure type] from [Author (Year)]. Extract the key techniques into a reusable template and append it to `graph-templates.md`."

The AI will:
1. Identify non-obvious techniques buried in the code
2. Strip paper-specific variable names and generalize
3. Annotate each non-standard option with an explanation
4. Append the template to `graph-templates.md`

**Example:** The figure below is from Huang & Zhang (2021), *AEJ: Applied*, showing heterogeneity coefficients by gender, education, and income level — a horizontal coefficient plot with three labeled sub-panels.

> Save the paper figure as `assets/figure3_panel_b.png` to display it here:
> `![Huang & Zhang 2021, Figure 3 Panel B](assets/figure3_panel_b.png)`

From the replication code, the AI extracted the following techniques into `graph-templates.md` (Section 6A):

| Technique | Explanation |
|-----------|-------------|
| `replace n = -n` | Negating the row index makes items display top-to-bottom as listed; without this, row 1 appears at the bottom |
| Empty rows as spacers | Rows with missing `coef`/`se` create blank gaps between panels; the same rows anchor `text()` panel headers |
| `xscale(alt)` | Moves the x-axis from its default bottom position to the top of the horizontal chart |
| `yscale(off)` + `ylabel(... , noticks)` | Suppresses all numeric y-axis labels while preserving coordinate structure for `text()` |
| `mlabp(12)` | Places group labels at the 12 o'clock position (above the marker) |
| `text(y x "...", place(e))` | Panel headers anchored to spacer row y-positions and the leftmost x value |

#### With image only (no source code)

Even without replication code, the AI can reconstruct a reasonable approach using the skill:

- Identify the chart type from the figure (horizontal coefficient plot, event study, RD, etc.)
- Route to relevant reference files (`estimation-to-graph.md`, `markers.md`, `axes.md`, etc.)
- Propose a plausible implementation based on the option inventories in those files

**Limitation:** Without source code, the AI cannot infer non-visual implementation details:
- Data construction tricks (e.g., the `replace n = -n` index flip)
- Exact numeric parameters (xlabel range, `text()` coordinates)
- Presence of rare options (e.g., `xscale(alt)`)

Image + source code together yield more precise, generalizable templates than either alone.

### Build Status

| File | Status | Description |
|------|--------|-------------|
| `estimation-to-graph.md` | ✅ | Bridge: regression results → plottable data |
| `graph-templates.md` | ✅ | Ready-to-run templates (event study, coefplot, margins, heterogeneity, etc.); grows via `/learn` |
| `ci-bands.md` | ✅ | rcap / rspike / rarea comparison + event study template |
| `twoway-syntax.md` | ✅ | All twoway plottypes (43), overlay syntax, twoway_options, addplot(), recast() |
| `axes.md` | ✅ | xlabel/ylabel, xscale/yscale, reference lines, dual y-axis |
| `schemes-regions.md` | ✅ | Scheme comparison, graph regions, graph size, publication style |
| `markers.md` | ✅ | msymbol (all symbolstyles), msize, mcolor/mfcolor/mlcolor, markerstyle presets |
| `lines.md` | ✅ | lpattern (named + formula), lwidth, lcolor, lstyle, connect_options |
| `colors.md` | ✅ | Named colors, RGB/CMYK/HSV/hex, opacity (%), intensity (*), palettes package |
| `labels-text.md` | ✅ | title/subtitle/note/caption, text()/ttext(), textbox_options, mlabel/mlab* |
| `legend.md` | ✅ | legend() contents vs location, order/label/lastlabel, by()-specific rules |
| `export-combine.md` | ✅ | saving/use/display workflow, graph combine alignment, graph export formats |
| `graph-bar-box.md` | ✅ | Lightweight guide to bar/box/dot graphs for common economics use cases |
| `by-over.md` | ✅ | Decision rules for over() vs by() vs graph combine |
| `SKILL.md` | ✅ | Routing table: task type → reference files, 10 high-frequency pitfalls, stata skill integration |

Status: ⬜ Not started · 🔄 In progress · ✅ Complete

### Reference Sources

- **Mitchell (2022)** — *A Visual Guide to Stata Graphics*, 4th ed. (Stata Press)
- **Stata Graphics Reference Manual `[G]`** — bundled with Stata; access via `help graph`
- **[dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)** — source for the bridge file

### Contributing

This skill improves through use:
1. Fix or extend the relevant reference file
2. Add new figure templates from real papers: provide the figure (and code if available) to the AI and direct it to append to `graph-templates.md`
3. Note the source (author, journal, year) and Stata version
4. Open a PR or issue
