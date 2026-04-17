# Stata Graphics Skill - Routing Table

Read this file first for any Stata graphics task. Use the routing table below to identify which reference files to consult before writing code. Do not write graphics code from memory alone.

All reference files are in `~/.claude/stata-graphics/references/`.

---

## Scheme Setup

Set an explicit scheme at the top of graphics code to ensure reproducible appearance. `plotplain` (from `blindschemes`) is a common choice for economics publications:

```stata
set scheme plotplain   // ssc install blindschemes, replace
```

Use the scheme the user prefers. If no scheme is specified, use Stata's built-in `s1color`.

When a scheme is set, do **not** manually specify colors for individual graph elements (`xline`, `yline`, `rcap`, scatter markers, line plots, etc.) unless the user explicitly requests a specific color. The scheme provides appropriate visual defaults for all elements.

---

## When This Skill Applies

Use this skill whenever the task involves:

- Creating any Stata figure, plot, or chart
- Modifying graph style, axes, colors, markers, or layout
- Exporting or combining graphs
- Visualizing estimation results (event study, coefficients, margins, DiD, RD, etc.)

Do **not** route to this skill for pure regression analysis, data cleaning, or table output (`esttab`, `outreg2`) — use `~/.claude/stata/SKILL.md` for those tasks.

---

## Execution Protocol

Before writing any graphics code, follow these two steps in order:

**Step 1 — Template first**: read `estimation-to-graph.md`, then `graph-templates.md`, and identify the closest matching template for the task. Do not write any code at this stage. For descriptive tasks (scatter, trend lines, distributions, bar/box charts), skip to the Routing Table directly.

**Step 2 — Fill in details**: read the remaining files specified in the routing table below, then adapt the template to the specific task requirements.

Do not skip Step 1. Do not write code from memory alone.

---

## Routing Table

### Estimation Result Visualization

For all estimation figures, start by reading `estimation-to-graph.md`, then `graph-templates.md`. They cover result extraction workflows and ready-to-run code templates. Then add task-specific files as needed.

| Task | Read These Files |
|------|-----------------|
| Event study / parallel trends | `estimation-to-graph.md` → `graph-templates.md` → `ci-bands.md` → `axes.md` |
| Coefficient plot — single model | `estimation-to-graph.md` → `graph-templates.md` → `markers.md` → `lines.md` |
| Coefficient plot — multi-model comparison | `estimation-to-graph.md` → `graph-templates.md` → `markers.md` → `lines.md` → `colors.md` |
| Heterogeneity / multi-panel figure | `estimation-to-graph.md` → `graph-templates.md` → `by-over.md` → `export-combine.md` |
| Marginal effects (`margins` / `marginsplot`) | `estimation-to-graph.md` → `graph-templates.md` |
| DiD dynamic effects (`csdid_plot`) | `estimation-to-graph.md` → `graph-templates.md` |
| Permutation / placebo density overlay | `estimation-to-graph.md` → `graph-templates.md` → `twoway-syntax.md` |
| RD plot | `estimation-to-graph.md` → `graph-templates.md` → `ci-bands.md` |

### Descriptive and Data Visualization

| Task | Read These Files |
|------|-----------------|
| Trend lines / time series | `twoway-syntax.md` → `axes.md` → `lines.md` |
| Scatter plot (with or without fit line / CI) | `twoway-syntax.md` → `markers.md` → `lines.md` |
| Distribution (`kdensity` / `histogram`) | `twoway-syntax.md` → `axes.md` |
| Bar chart / box plot / dot plot | `graph-bar-box.md` → `by-over.md` |
| Any twoway overlay involving CI bands | `ci-bands.md` → `twoway-syntax.md` |

### Style and Formatting (read on demand)

Read these only when the task requires specific style control — do not read all by default.

| Need | File |
|------|------|
| Overall scheme / publication style | `schemes-regions.md` |
| Graph size, plot region, margins | `schemes-regions.md` |
| Colors, opacity, palettes | `colors.md` |
| Marker symbols, size, fill | `markers.md` |
| Line pattern, width, connect mode | `lines.md` |
| Axis labels, ticks, scale, dual y-axis | `axes.md` |
| Titles, added text, data point labels | `labels-text.md` |
| Legend position, order, contents | `legend.md` |
| Multi-panel layout (`by()` / `over()` / `graph combine`) | `by-over.md` |
| Export format, `graph combine` alignment | `export-combine.md` |

---

## High-Frequency Pitfalls

These errors recur across task types. Check them before finalizing any code.

1. **Variable order in `twoway`**: always `y x` — write `scatter yvar xvar`, never reversed.

2. **`rcap` CI order**: `rcap hi_var lo_var x_var` — upper bound first, lower bound second. Reversed order produces inverted caps.

3. **`yaxis()`/`xaxis()` are plot-level options**: they go inside the plottype specification (`scatter y x, yaxis(2)`), not at the `twoway` level. Placing them at the `twoway` level causes an error or is silently ignored.

4. **`range()` only expands**: `xscale(range(0 10))` cannot shrink the displayed range below the data extent. To restrict the visible range, filter the data with an `if` condition instead.

5. **`xlabel(..., add)`**: without `add`, custom labels silently replace all existing default labels. Use `add` to append custom ticks to the existing set.

6. **`||` option scope**: options written after `||` apply only to the immediately preceding plot, not to all plots in the overlay. Use `twoway`-level options for global settings.

7. **`by()` vs `graph combine`**: `by()` is built-in faceting for the same graph type with uniform scale; `graph combine` assembles heterogeneous panels saved as separate graphs. Mixing them up produces wrong layouts or errors.

8. **`over()` is not a panel mechanism**: it groups categories within a single graph. For multi-panel layouts use `by()` or `graph combine`.

9. **`scheme()` per-graph overrides `set scheme`**: if the global scheme is set but the output looks wrong, check for a conflicting graph-level `scheme()` option.

10. **`graph export` format is determined by file extension**: `graph export fig.pdf` → PDF; `graph export fig.png` → PNG. Always include the extension explicitly.

---

## Integration with the Stata Skill

These two skills cover complementary stages of the same empirical workflow:

```
Regression analysis   →  ~/.claude/stata/SKILL.md
Result extraction     →  estimation-to-graph.md   (this skill)
Figure production     →  this skill's references
```

For tasks that span both regression and plotting, read both SKILL.md files. Any task involving extraction of `e()` stored results for plotting should always start with `estimation-to-graph.md`.

Do **not** route graphics tasks to `~/.claude/stata/references/graphics.md` or `~/.claude/stata/packages/graph-schemes.md` — those files are superseded by this skill for all graphics work.
