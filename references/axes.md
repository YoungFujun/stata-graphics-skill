# axes.md — Axis Control in Stata

Source: Mitchell (2022) Ch.8; Stata 19 Graphics Reference Manual [G-3]
`axis_choice_options` (pp.472-476), `axis_label_options` (pp.477-493),
`axis_options` (pp.494-495), `axis_scale_options` (pp.496-503),
`axis_title_options` (pp.504-508).

---

## 1. Overview: Four Option Groups

All axis options are **twoway-level** — placed after the last plot parenthesis.
Exception: `yaxis()`/`xaxis()` are **plot-level** options (see Section 4).

| Option group | Controls |
|--------------|---------|
| `axis_label_options` | Tick placement and labels (`xlabel`, `ylabel`, etc.) |
| `axis_scale_options` | Scale type, range, and axis line look (`xscale`, `yscale`) |
| `axis_title_options` | Axis title text (`xtitle`, `ytitle`) |
| `axis_choice_options` | Which axis a plot uses when multiple axes exist — **plot-level** |

`xline()` / `yline()` (reference lines) are also twoway-level options; see Section 5.

---

## 2. axis_label_options — Ticks and Labels

### 2.1 Syntax

```stata
{y|x|t|z}{label|tick|mlabel|mtick}(rule_or_values [, suboptions])
```

| Option | Effect |
|--------|--------|
| `ylabel()` / `xlabel()` | Major ticks + labels |
| `ytick()` / `xtick()` | Major ticks only (no labels) |
| `ymlabel()` / `xmlabel()` | Minor ticks + labels |
| `ymtick()` / `xmtick()` | Minor ticks only |

### 2.2 Rules

| Rule | Example | Description |
|------|---------|-------------|
| `##` | `#6` | Ask Stata to choose ~6 "nice" values |
| `###` | `##10` | Place 9 minor ticks between major ticks (mlabel/mtick only) |
| `#(#)#` | `-4(.5)3` | From -4 to 3 in steps of 0.5 |
| `minmax` | `minmax` | Label only the minimum and maximum values |
| `none` | `none` | No labels and no ticks |
| `.` | `.` | Skip the rule (same as `none` unless combined with `add`) |

You can also specify a numlist directly: `ylabel(0 5 10 25 50)`
or combine a rule with a numlist: `ylabel(0(5)50 75 100)`.

### 2.3 Text substitution for labels

```stata
// Replace numeric values with custom text
xlabel(1 "Jan" 2 "Feb" 3 "Mar" 4 "Apr" 5 "May" 6 "Jun" ///
       7 "Jul" 8 "Aug" 9 "Sep" 10 "Oct" 11 "Nov" 12 "Dec")

// Event study time axis (t = -1 is the omitted period)
xlabel(-4 "-4" -3 "-3" -2 "-2" -1 "Pre" 0 "Event" 1 "1" 2 "2" 3 "3")
```

### 2.4 Key suboptions

| Suboption | Description |
|-----------|-------------|
| `axis(#)` | Which axis this applies to (for multi-axis graphs) |
| `add` | **Append** to existing ticks/labels rather than replace them; see Trap 1 |
| `noticks` / `ticks` | Suppress / force ticks |
| `nolabels` / `labels` | Suppress / force label text |
| `valuelabel` | Map values through the variable's value label (automatic text substitution) |
| `format(%fmt)` | Format numeric values (e.g. `format(%9.2f)`, `format(%td)`) |
| `angle(anglestyle)` | Rotate labels: `angle(0)` horizontal, `angle(90)` vertical, `angle(45)` |
| `alternate` | Offset adjacent labels to prevent overlap (useful for dense x-axis) |
| `norescale` | Add ticks/labels outside the current axis range without rescaling the axis |
| `labsize(textsizestyle)` | Label font size (e.g. `labsize(small)`, `labsize(vsmall)`) |
| `labcolor(colorstyle)` | Label color |
| `labgap(size)` | Gap between tick and label |
| `labstyle(textstyle)` | Overall label text style |
| `labelminlen(#)` | Minimum label width in characters (pad with spaces; useful for aligning combined graphs) |
| `tlength(size)` | Tick length |
| `tposition(outside\|crossing\|inside)` | Tick direction (default: `outside`) |
| `tlstyle()` / `tlwidth()` / `tlcolor()` | Tick line style / thickness / color |
| `custom` | Apply rendition suboptions only to labels added by the current `add` call |
| `grid` / `nogrid` | Draw / suppress grid lines at major tick positions |
| `gmin` / `gmax` / `nogmin` / `nogmax` | Control grid lines at the min/max tick values |
| `gstyle(gridstyle)` | Overall grid style |
| `gextend` / `nogextend` | Extend grid lines into the plot region margin |
| `glstyle()` / `glwidth()` / `glcolor()` / `glpattern()` | Grid line style attributes |
| `tstyle(tickstyle)` | Overall tick + label style |

### 2.5 Examples

```stata
// Approximately 5 ticks (Stata chooses values)
scatter y x, ylabel(#5)

// Exact range and step
scatter y x, ylabel(-0.2(0.1)0.4)

// Format only — keep Stata's auto-chosen values
scatter y x, ylabel(, format(%9.2f))

// Formatted + angled (useful for large numbers)
scatter y x, ylabel(, format(%12.0gc) angle(0))

// Stagger x-axis labels to avoid overlap
scatter y x, xlabel(, alternate)

// Map a numeric variable's value label to the axis
scatter y cat_var, xlabel(1 2 3, valuelabel)

// Append a special label without removing defaults
scatter y x, ylabel(0 "Ref", add custom labcolor(red))

// Grid lines on y-axis
twoway line y x, ylabel(, grid glcolor(gs14) glpattern(solid))
```

### 2.6 Practical heuristics from Mitchell

Mitchell's value here is mostly design guidance rather than extra syntax:

- When an axis needs many labels, first reduce density if possible. If the labels must stay dense, `angle(45)` is often the cleanest compromise:
  ```stata
  scatter workers2 faminc, xlabel(15000(1000)30000, angle(45))
  ```
- Use `angle(90)` more cautiously. It saves space, but usually slows reading compared with horizontal or 45-degree labels.
- For publication figures, axis titles should normally be visually larger than axis labels. If you need exact sizing, coordinate `xtitle()/ytitle()` and `xlabel()/ylabel(), labsize()`; see [schemes-regions.md](./schemes-regions.md) for sizing strategy.

---

## 3. axis_scale_options — Scale, Range, and Axis Line

### 3.1 Syntax

```stata
yscale(axis_suboptions)
xscale(axis_suboptions)
tscale(axis_suboptions)    // time axis
```

Abbreviations: `ysc()` / `xsc()`; `range()` can be abbreviated `r()`.

### 3.2 Suboptions

| Suboption | Description |
|-----------|-------------|
| `axis(#)` | Which axis to modify (multi-axis graphs) |
| `log` / `nolog` | Natural-log scale; axis labels show original units |
| `reverse` / `noreverse` | Run scale from max to min |
| `range(numlist)` | Expand axis range to include specified values; **never narrows** — see Trap 2 |
| `off` / `on` | Suppress / force the entire axis (line, ticks, labels, title all hidden) |
| `noline` / `line` | Hide / force the axis line while keeping ticks, labels, and title |
| `fill` | Allocate space for the axis even when `off` is specified |
| `alt` | Move axis to opposite side (y-axis → right; x-axis → top) |
| `fextend` | Extend axis line through plot region and its margins (default for most schemes) |
| `extend` | Extend axis line through plot region only |
| `noextend` | Do not extend axis line beyond data range |
| `titlegap(size)` | Margin between axis title and tick labels |
| `outergap(size)` | Margin outside the axis title |
| `lstyle(linestyle)` | Overall axis line style |
| `lcolor(colorstyle)` | Axis line color |
| `lwidth(linewidthstyle)` | Axis line thickness |
| `lpattern(linepatternstyle)` | Axis line pattern |

### 3.3 Examples

```stata
// Log scale (labels show original values)
scatter y x, xscale(log)

// Reversed y-axis (e.g. rank: 1 at top)
scatter rank x, yscale(reverse)

// Ensure y-axis includes 0 (does not clip data)
scatter y x, yscale(range(0))

// Hide x-axis entirely
scatter y x, xscale(off)

// Hide axis line but keep ticks and labels
scatter y x, yscale(noline)

// Move y-axis to right side
scatter y x, yscale(alt)

// Reduce gap between tick labels and axis title
scatter y x, yscale(titlegap(1))
```

---

## 4. axis_title_options — Axis Titles

### 4.1 Syntax

```stata
ytitle("string" ["string" ...] [, suboptions])
xtitle("string" ["string" ...] [, suboptions])
ttitle(...)    // synonym for xtitle() on time axes
```

`string` supports Unicode characters and SMCL tags for math symbols, italics, etc.

### 4.2 Suboptions

| Suboption | Description |
|-----------|-------------|
| `axis(#)` | Which axis this title belongs to |
| `prefix` | Prepend text to an existing title |
| `suffix` | Append text to an existing title |
| *textbox_options* | Control text appearance (color, size, margin, etc.); see [G-3] `textbox_options` |

### 4.3 Examples

```stata
// Basic
scatter y x, ytitle("Coefficient") xtitle("Event time (quarters)")

// Multi-line title (one string per line)
scatter y x, ytitle("Estimated" "treatment effect")

// Suppress axis title
scatter y x, ytitle("")

// Independent titles for each y-axis in a dual-axis graph
twoway (scatter gnp year, yaxis(1)) ///
       (scatter r year, yaxis(2)), ///
       ytitle("GNP (left)", axis(1)) ///
       ytitle("Interest rate (right)", axis(2))

// Suppress the second y-axis title
twoway ..., ytitle("", axis(2))

// SMCL: Greek letters and subscripts
scatter y x, ytitle("{&beta}{subscript:t}")
```

---

## 5. axis_choice_options — Assigning Plots to Axes (⚠ plot-level)

### 5.1 Critical rule: yaxis() / xaxis() are plot-level options

They must go **inside** each individual plot's parentheses, not at the twoway level.

```stata
// ❌ Wrong: yaxis(2) placed at the twoway level
twoway (scatter gnp year) (scatter r year), yaxis(2)

// ✅ Correct: yaxis(2) inside the relevant plot's parentheses
twoway (scatter gnp year, yaxis(1)) (scatter r year, yaxis(2))
// yaxis(1) is the default, so this also works:
twoway (scatter gnp year) (scatter r year, yaxis(2))
```

### 5.2 Syntax

```stata
yaxis(# [#...])    // 1 ≤ # ≤ 9
xaxis(# [#...])    // 1 ≤ # ≤ 9
```

### 5.3 Two uses of dual y-axes

**Use A: Two variables on different scales**

```stata
twoway (scatter gnp year) (scatter r year, yaxis(2)), ///
       ytitle("GNP", axis(1)) ytitle("Rate (%)", axis(2)) ///
       ylabel(#5, axis(1)) ylabel(#5, axis(2))
```

**Use B: Shared scale, but label a special value on the opposite axis**

```stata
// yaxis(1 2): creates mirror axes sharing the same scale
// Useful for annotating a specific value on the right-hand axis
scatter bp concentration, yaxis(1 2) ylabel(120, axis(2) add)
```

### 5.4 Setting axis options for each axis independently

```stata
twoway (scatter gnp year) (scatter r year, yaxis(2)), ///
       ylabel(#5, axis(1))         ///
       ylabel(0 5 10 15, axis(2))  ///
       ylabel(#5, axis(1))
```

### 5.5 Notes

- Each individual plot may have at most one x scale and one y scale.
- Three or more y-axes stack on the same side and become difficult to read; avoid if possible.
- In dual-axis graphs, keep axis targeting consistent across all related options. If a plot uses `xaxis(2)` or `yaxis(2)`, any axis-specific labels, titles, reference lines, or annotations tied to that plot should usually target the same axis number.

---

## 6. xline() / yline() — Reference Lines

These are **twoway-level options** (not part of the axis_* family),
placed at the end of the command outside all plot parentheses.

### 6.1 Syntax

```stata
yline(numlist [, suboptions])
xline(numlist [, suboptions])
```

| Suboption | Description |
|-----------|-------------|
| `lstyle(linestyle)` | Overall line style |
| `lcolor(colorstyle)` | Color |
| `lwidth(linewidthstyle)` | Thickness |
| `lpattern(linepatternstyle)` | Pattern (solid / dash / dot / shortdash, etc.) |
| `axis(#)` | Which axis the reference line belongs to (dual-axis graphs) |

### 6.2 Examples

```stata
// Zero reference line (standard for event study)
twoway ..., yline(0, lcolor(gray) lpattern(dash))

// Treatment-period marker
twoway ..., xline(0, lcolor(black) lwidth(medthin))

// Both axes
twoway ..., yline(0, lcolor(gray)) xline(0, lcolor(gray))

// Reference line on the second y-axis
twoway ..., yline(0, axis(2) lcolor(red) lpattern(dash))

// Multiple reference lines
twoway ..., xline(-4 0 4, lcolor(gray) lpattern(shortdash))
```

---

## 7. Common Traps

### Trap 1 — Missing `add` wipes out default labels

```stata
// Goal: add a label at 0 without removing the auto-chosen ticks
// ❌ Wrong: ylabel(0) clears all defaults; only 0 is labeled
scatter y x, ylabel(0)

// ✅ Correct: use add to append
scatter y x, ylabel(0, add)

// ✅ More common: specify the main rule, then append the special value
scatter y x, ylabel(#5) ylabel(0 "Ref", add custom labcolor(red))
```

### Trap 2 — `range()` only expands, never narrows

```stata
// ❌ Incorrect expectation: show only x between 10 and 50
scatter y x, xscale(range(10 50))   // does not clip data; only expands range

// ✅ Correct: filter data with if
scatter y x if x >= 10 & x <= 50
```

### Trap 3 — `yaxis()` / `xaxis()` must be plot-level, not twoway-level

```stata
// ❌ Wrong
twoway (scatter y1 x) (scatter y2 x), yaxis(2)

// ✅ Correct
twoway (scatter y1 x) (scatter y2 x, yaxis(2))
```

### Trap 4 — Minor ticks: `##N` means N−1 intervals, not N

```stata
// ##5 produces 4 minor ticks between each pair of major ticks (not 5)
// Stata interprets the number as the division count, then subtracts 1
ymtick(##5)    // → 4 minor ticks per major interval
xmtick(##10)   // → 9 minor ticks (≈ tenths)

// For exact control, use #(#)# or an explicit numlist
```

### Trap 5 — axis_options belong to twoway, not to individual plottypes

```stata
// ✅ Correct placement: axis_options after all plot parentheses
twoway (scatter y x) (line y2 x), ylabel(#5) xtitle("Time")

// yaxis() / xaxis() are the only axis options that must go inside plot parentheses
```

### Trap 6 — `yscale(log)` vs. manually generating a log variable

```stata
// yscale(log): axis labels show original units (readable)
scatter lexp gnppc, xscale(log)     // x labels: 1000, 10000, ...

// gen log_x = log(x) then plot: labels show log values (6.9, 9.2, ...)
// Readers must back-transform mentally — usually inferior to xscale(log)
```

### Trap 7 — Dense labels: rotating should not be your first reflex

```stata
// Often better: reduce label count first
twoway line y date, tlabel(01jan2010(2)01jan2020)

// If dense labeling is substantively necessary, then rotate
twoway line y date, tlabel(01jan2010(1)01jan2020, angle(45))
```

Mitchell's examples consistently treat rotation as a readability fix, not as a substitute for selecting fewer, better labels.

---

## 8. Publication Templates

### Event study (full axis setup)

```stata
twoway (rcap hi lo time, lcolor(navy) lwidth(medium)) ///
       (scatter coef time, mcolor(navy) msize(medium) msymbol(circle)), ///
       yline(0, lcolor(gray) lpattern(dash) lwidth(thin)) ///
       xline(-1, lcolor(black) lwidth(thin) lpattern(shortdash)) ///
       xlabel(-6(1)6) ///
       ylabel(-0.2(0.1)0.2, format(%9.2f) angle(0)) ///
       ytitle("Estimated coefficient") ///
       xtitle("Years relative to treatment") ///
       legend(off)
```

### Dual y-axis (coefficient + density)

```stata
twoway (scatter coef year, yaxis(1) mcolor(navy)) ///
       (kdensity pvalue, yaxis(2) lcolor(maroon) lpattern(dash)), ///
       ylabel(#5, axis(1)) ///
       ylabel(0(0.5)2, axis(2)) ///
       ytitle("Coefficient", axis(1)) ///
       ytitle("Density", axis(2)) ///
       xtitle("Year") ///
       yline(0, axis(1) lcolor(gray) lpattern(shortdash))
```

### Time-series with date x-axis

```stata
twoway line y date, ///
       tlabel(01jan2010(1)01jan2020, format("%tdMon-YY") angle(45)) ///
       ytitle("Index") xtitle("") ///
       yscale(range(0))
```
