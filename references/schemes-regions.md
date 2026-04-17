# schemes-regions.md — Graph Schemes and Region Control in Stata

Source: Stata 19 Graphics Reference Manual [G-3] `region_options` (pp.623–634),
[G-4] `Schemes intro` (pp.739–747), individual scheme entries (pp.748–758);
Mitchell (2022) §9.2–§9.4; `~/.claude/stata/packages/graph-schemes.md`.

---

## 1. Graph Schemes — Overview

A **scheme** is a set of default visual attributes (background colors, marker shapes,
line patterns, text sizes, legend placement, etc.) that applies to all graphs
until changed.

```stata
// Change scheme for current session only
set scheme plotplain

// Change scheme permanently (persists across Stata restarts)
set scheme plotplain, permanently

// Check current scheme setting
query graphics

// Apply to a single graph only
scatter y x, scheme(economist)

// List all installed schemes
graph query, schemes

// Redisplay a saved graph with a different scheme
graph use mygraph, scheme(plotplain)
```

**Three scope levels (narrowest wins):**

| Level | Syntax | Scope |
|-------|--------|-------|
| Permanent | `set scheme name, permanently` | Survives restarts |
| Session | `set scheme name` | Until next `set scheme` or restart |
| Graph-specific | `scheme(name)` option | Single graph only |

**Important:** Scheme is applied at *display* time, not save time. A graph saved
to `.gph` can be redisplayed with a different scheme.

---

## 2. Built-in Stata Schemes

### 2.1 Complete Official Scheme List

| Scheme | Foreground | Background | Notes |
|--------|-----------|-----------|-------|
| **stcolor** | Color | White | **Factory default since Stata 18** |
| stmono1 | Gray scale | White | Monochrome variant |
| stmono2 | Gray scale | White | Alternative monochrome |
| stgcolor | Color | Gray | Dark background version |
| stcolor_alt | Color | White | Alternate color ordering |
| stsj | Color | White | Stata Journal style |
| **s2color** | Color | White | Prior default (Stata 17 and earlier) |
| s2mono | Gray scale | White | s2 family monochrome |
| s2gcolor | Color | Light gray | s2 family, gray plot region |
| s2manual | — | — | Used in s2 documentation |
| s2gmanual | — | — | Used in s2 gray documentation |
| s2color8 | Color | White | Pre–Stata 8 color ordering |
| **s1color** | Color | White | Clean, no background tinting |
| s1mono | Gray scale | White | Clean monochrome |
| s1rcolor | Color | Black | Reverse (dark) background |
| s1manual | — | — | Used in s1 documentation |
| economist | Color | Light blue | Economist-style (y-axis on right) |
| sj | B&W | White | Stata Journal official style |

**Stata 18+ note:** `stcolor` replaced `s2color` as the factory default.
Code relying on `s2color` defaults may look different on Stata 18+.
To restore pre-18 appearance: `set scheme s2color`.

### 2.2 Scheme Visual Characteristics

| Scheme | Plot region | Graph region | Legend | y-axis | Notes |
|--------|------------|-------------|--------|--------|-------|
| s2color | White | Light blue tint | Bottom | Rotated 90° | Prior standard; colored lines |
| s1color | White | White | — | — | No tinting; clean; different colors from s2color |
| s2mono | White | Light gray | — | — | Varying symbol shapes instead of colors |
| s1mono | White | White | — | — | Clean monochrome |
| economist | Light bg | Colored bg | Top | **Right side** | Left-justified titles; **not suitable for CI regions** |
| plotplain | White | White | Right | Unrotated | No shading; faint gridlines; small hollow markers |
| plottig | Gray | White | Right | — | White grid lines on gray plot region |
| 538 | Gray | Gray | — | — | Dark gray grid; colorful markers |
| 538w | White | White | — | — | 538 palette on white background |
| lean1 | White | White | — | — | No grid; framed; hollow/solid circles |
| lean2 | White | White | — | — | Horizontal gridlines only; no frame |

---

## 3. Community Schemes

### 3.1 blindschemes

```stata
ssc install blindschemes, replace
```

| Scheme | Description |
|--------|-------------|
| `plotplain` | Minimalist: white background, black lines, no gridlines — **most common for economics publications** |
| `plotplainblind` | plotplain + Okabe & Ito colorblind-safe palette |
| `plottig` | Tufte-inspired with colorblind-safe colors |
| `plottigblind` | Enhanced colorblind version of plottig |

### 3.2 g538schemes

```stata
ssc install g538schemes, replace
```

| Scheme | Description |
|--------|-------------|
| `538` | FiveThirtyEight style: gray background everywhere, colorful markers |
| `538w` | 538 palette on white background |
| `538bw` | 538 black-and-white version |

### 3.3 lean schemes

```stata
net install gr0002_3, from(http://www.stata-journal.com/software/sj4-3)
```

| Scheme | Description |
|--------|-------------|
| `lean1` | White; no grid; framed; hollow and solid circles |
| `lean2` | White; horizontal gridlines; no frame |

### 3.4 schemepack

```stata
ssc install schemepack, replace
```

Selected schemes: `tableau`, `gg_s2color` (ggplot2-inspired), `tufte` (Tufte minimalism),
`burd` (minimalist), `cblind1` (8-color colorblind-safe).

---

## 4. grstyle — Dynamic Graph Styling

`grstyle` (`ssc install grstyle`) modifies scheme defaults on the fly without
writing a `.scheme` file. Changes apply to all subsequent graphs in the session
until cleared. Requires `palettes` and `colrspace` as dependencies.

```stata
ssc install grstyle, replace
ssc install palettes, replace
ssc install colrspace, replace

grstyle init              // must run before any grstyle set
grstyle set plain         // white background, clean style
grstyle set plain, box    // + framing box

grstyle set color tableau          // Tableau palette
grstyle set color "0 114 178" "213 94 0"  // custom RGB
grstyle set symbol O D T S         // marker symbols
grstyle set lpattern solid dash dot
grstyle set size 10pt              // all text
grstyle set size axis_title: 11pt  // specific element

grstyle clear             // remove all grstyle settings
```

**Scope note:** `grstyle` settings are session-global and stack on top of the
active scheme. Run `grstyle clear` before changing schemes.

See `~/.claude/stata/packages/graph-schemes.md` for full `grstyle` documentation
and `colorpalette` usage.

---

## 5. Custom Scheme Creation

Create a `.scheme` text file that inherits from an existing scheme:

```stata
// File: scheme-mypaper.scheme
// Location: ~/ado/plus/s/  (find with: sysdir)

#include s2color

color background white
color plotregion white
linewidth p medthick
gsize axis_title medsmall
gsize tick_label small
```

```stata
// Verify it is visible
graph query, schemes    // should list mypaper

// Use it
set scheme mypaper
```

**Finding the PERSONAL ado directory:** `sysdir` shows the path.
Save `.scheme` files to the `PERSONAL` path (e.g., `~/ado/personal/s/`).

---

## 6. Publication Recommendations — Economics Journals

### Quick recommendation

| Use case | Recommended scheme |
|----------|--------------------|
| Economics working paper / journal submission | `plotplain` (blindschemes) |
| Monochrome print | `s1mono` or `plotplain` + grayscale colors |
| Colorblind-accessible | `plotplainblind` |
| Stata Journal submission | `sj` |
| Slides / presentation | `538` or `tableau` |
| **Avoid** | `economist` (y-axis on right; unusual layout for economics papers) |

### Publication-style template

```stata
set scheme plotplain   // scheme provides color defaults; omit per-element colors

twoway ///
    (rcap ci_hi ci_lo event_time, msize(small)) ///
    (scatter coef_b event_time, msymbol(circle) msize(small)) ///
    , ///
    yline(0, lwidth(thin)) ///
    xline(-0.5, lpattern(dash) lwidth(thin)) ///
    xlabel(-6(1)6) ylabel(, format(%9.2f) angle(0)) ///
    ytitle("Coefficient") xtitle("Event time (years)") ///
    graphregion(color(white) margin(medium)) ///
    plotregion(lcolor(black) margin(zero)) ///
    legend(off)

graph export "fig_event_study.pdf", replace
```

---

## 7. Graph Regions — Four-Layer Nesting

Stata graphs have four nested regions (outermost to innermost):

```
┌─────────────────────────────────────────────────────┐
│  Available area  (xsize × ysize)                    │
│  ┌───────────────────────────────────────────────┐  │
│  │  Outer graph region  (graphregion margin)     │  │
│  │  [titles, notes placed here]                  │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │  Outer plot region  (plotregion border) │  │  │
│  │  │  [axes drawn on/near this border]       │  │  │
│  │  │  ┌───────────────────────────────────┐  │  │  │
│  │  │  │  Inner plot region (data area)    │  │  │  │
│  │  │  │  [plotregion margin controls      │  │  │  │
│  │  │  │   offset between axis and data]   │  │  │  │
│  │  │  └───────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

**Construction order:** available area → `graphregion(margin())` shrinks for
title/note placement → axes drawn on outer plot region border → `plotregion(margin())`
creates offset between axis line and data → inner plot region holds data.

**Fill color semantics:**

| Option | What it fills |
|--------|---------------|
| `graphregion(fcolor(color))` | Outer graph region (the "background" visible around axes) |
| `plotregion(fcolor(color))` | Outer plot region (area bounded by axis lines) |
| `plotregion(ifcolor(color))` | Inner plot region only (rarely used separately) |
| `graphregion(ifcolor(color))` | Inner graph region (little practical use) |

Most publication use cases: set `graphregion(color(white))` and `plotregion(lcolor(black) margin(zero))`.

---

## 8. region_options — Syntax Reference

### 8.1 Overall size: ysize() / xsize()

```stata
ysize(#[units])     // graph height; default: 4 in
xsize(#[units])     // graph width;  default: 5.5 in (scheme-dependent)
```

Units: `in` (default), `pt` (1/72 in), `cm` (2.54 cm = 1 in).
Range: 1 in to 100 in. These control the *available area*, not the plot region itself.

```stata
scatter y x, xsize(6.5) ysize(4)    // full-page width (typical journal)
scatter y x, xsize(3.25) ysize(3)   // two-column format
```

**Resize an already-drawn graph in memory:**

```stata
graph display, xsize(6.5) ysize(4)
```

### 8.2 graphregion() / plotregion()

Both accept the same suboptions. Options for `graphregion()` affect the outer
region; options for `plotregion()` affect the inner region.

```stata
graphregion(suboptions)
plotregion(suboptions)
```

| Suboption | Description |
|-----------|-------------|
| `color(colorstyle)` | Outer (background) region color and fill |
| `fcolor(colorstyle)` | Fill color only (separates fill from outline) |
| `lcolor(colorstyle)` | Border line color |
| `lwidth(linewidthstyle)` | Border line thickness |
| `lpattern(linepatternstyle)` | Border line pattern |
| `lstyle(linestyle)` | Overall border line style |
| `margin(marginstyle)` | Margin between this region and the next region inward |
| `style(areastyle)` | Overall area style |
| `istyle(areastyle)` | Inner area style |
| `icolor(colorstyle)` | Inner region color |
| `ifcolor(colorstyle)` | Inner region fill color |
| `ilcolor(colorstyle)` | Inner region border color |
| `ilwidth(linewidthstyle)` | Inner region border thickness |

**`margin()` values (margin styles):**

| Value | Effect |
|-------|--------|
| `zero` | No margin at all — axes touch the region boundary |
| `small` / `medsmall` / `medium` / `medlarge` / `large` / `vlarge` | Named sizes |
| `l=#` `r=#` `t=#` `b=#` | Set individual sides in points |
| `l+#` `r+#` `t+#` `b+#` | Add to current margin |
| `vsmall` | Very small margin |

```stata
// Remove offset between axis line and data (publication standard)
plotregion(margin(zero))

// Add extra whitespace on left and right of graph region
graphregion(margin(l+10 r+10))

// Suppress plot region border
plotregion(style(none))

// White plot background, thin black border, no data offset
plotregion(fcolor(white) lcolor(black) margin(zero))
```

### 8.3 aspectratio()

```stata
aspectratio(#)    // # = height/width of the plot region (not total graph)
```

`aspectratio(1)` = square plot region; `aspectratio(0.5)` = wide/flat;
`aspectratio(2)` = tall/narrow. Overrides the natural ratio implied by `ysize`/`xsize`.

### 8.4 scale()

```stata
scale(#)    // multiplies ALL text sizes, marker sizes, and line widths
```

Useful for small graphs where default text would be too large.
`scale(0.8)` reduces everything to 80%; `scale(1.2)` enlarges everything.

---

## 9. Absolute vs Relative Sizing Units

Stata accepts two categories of size specifications:

**Absolute units** — fix the physical size regardless of graph dimensions:
```stata
grstyle set size 10pt          // exact 10-point font
ytitle("", size(10pt))         // exact 10-point title
xsize(3.5in) ysize(2.5in)      // exact inches
```
Use when the final output size is precisely known (e.g., a specific journal's
figure format). Output will look wrong if the graph is later resized.

**Relative units** — scale proportionally with graph dimensions:
```stata
ytitle("", size(large))        // relative to scheme base size
ytitle("", size(*1.2))         // 1.2× the base size
ytitle("", size(5rs))          // 5 "relative size" units
```
Use when graphs may be resized (the usual case). Markers, lines, and text
stay visually consistent regardless of `xsize`/`ysize` values.

**Recommendation:** Default to relative units (`size(medsmall)`, `size(*1.2)`).
Switch to absolute units only when submitting to a publication with precise
size requirements.

---

## 10. Common Traps

### Trap 1 — Stata 18 default scheme change

```stata
// Code written for Stata ≤17 assumes s2color defaults.
// Running on Stata 18+ produces stcolor look instead.
// Fix: add explicit scheme() or set scheme at top of do-file
set scheme s2color    // restore prior-default look
```

### Trap 2 — economist scheme: y-axis on right

```stata
// economist moves the y-axis to the RIGHT side.
// Not suitable for standard economics figures.
// ❌ Don't use for CI plots, event studies, or most economics work
// ✅ plotplain is the standard choice
```

### Trap 3 — grstyle stacks; clear before scheme change

```stata
// grstyle settings accumulate within a session.
// If you change schemes without clearing, old grstyle settings persist.
grstyle clear         // always clear before switching
set scheme plotplain
grstyle init
```

### Trap 4 — plotregion(margin(zero)) vs yscale(range())

```stata
// plotregion(margin(zero)) removes the visual gap between
// the axis line and the closest data point.
// It does NOT change the axis range.
// Use yscale(range(0)) to extend the range; use margin(zero) for aesthetics.
```

### Trap 5 — color() vs fcolor() on regions

```stata
// color() sets BOTH fill and border to the same color.
// fcolor() sets fill only; border remains as-is.
// For publication: set fill and border separately for clarity.
graphregion(fcolor(white) lcolor(white))   // explicit: fill + border both white
```

---

## 11. Publication Templates

### Standard economics paper figure

```stata
set scheme plotplain   // scheme provides color defaults; omit per-element colors

twoway (scatter y x, msymbol(circle) msize(small)), ///
    graphregion(color(white)) ///
    plotregion(lcolor(black) margin(zero)) ///
    ytitle("Outcome") xtitle("Treatment intensity") ///
    ylabel(, format(%9.2f) angle(0)) ///
    xsize(4.5) ysize(3.5)

graph export "figure.pdf", replace
```

### Two-panel heterogeneity figure (graph combine)

```stata
set scheme plotplain

twoway ..., name(g1, replace) nodraw ///
    graphregion(color(white)) plotregion(lcolor(black) margin(zero))

twoway ..., name(g2, replace) nodraw ///
    graphregion(color(white)) plotregion(lcolor(black) margin(zero))

graph combine g1 g2, cols(2) ///
    graphregion(color(white)) ///
    xsize(6.5) ysize(3)

graph export "figure_hetero.pdf", replace
```

### Monochrome / grayscale (for print)

```stata
set scheme s1mono   // grayscale scheme; add lcolor() only if finer control needed

twoway ///
    (rcap ci_hi ci_lo t) ///
    (scatter coef t, msymbol(circle)), ///
    graphregion(color(white)) ///
    plotregion(lcolor(black) margin(zero)) ///
    yline(0, lpattern(dash))
```

### Quick session setup for an economics paper

```stata
set scheme plotplain
grstyle init
grstyle set plain, box
grstyle set color navy maroon forest_green
grstyle set size 10pt
grstyle set size axis_title: 11pt
```
