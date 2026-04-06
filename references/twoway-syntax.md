# twoway-syntax.md — Stata twoway Core Syntax Reference

Source: Stata 19 Graphics Reference Manual [G-2] graph twoway (p.201–208), scatter (p.403–424),
line (p.297–303), and all plottype entries; [G-3] twoway_options (p.668), addplot_option (p.458),
advanced_options (p.462). Session B2 of stata-graphics-skill.

---

## 1. Basic Syntax and the Plottype Concept

```stata
[graph] twoway plottype [varlist] [if] [in] [weight] [, options]
```

- Both `graph` and `twoway` prefixes are optional — for `scatter`, `line`, `connected`, etc., the plottype name alone is a valid command
- **A plottype is a subcommand**: `scatter`, `line`, `lfit`, etc. are all *plottypes* in the twoway family and also standalone commands
- **scatter is the mother of all twoway plottypes**: every twoway command ultimately reduces to scatter; scatter inherits all their options

```stata
. scatter y x               /* direct */
. twoway scatter y x        /* equivalent */
. graph twoway scatter y x  /* equivalent */
```

### Variable order convention

```stata
scatter y1 [y2 ...] x   /* last variable is x; all preceding variables are y */
```

Multiple y-variables is shorthand for multiple `||` overlays:
```stata
scatter y1 y2 x            /* equivalent to */
scatter y1 x || scatter y2 x
```

Options for multiple y-variables correspond positionally; unmatched variables use defaults:
```stata
scatter y1 y2 x, ms(O i) c(. l)  /* y1: msymbol(O), connect(.); y2: msymbol(i), connect(l) */
scatter y1 y2 x, ms(O)            /* y1: msymbol(O); y2: default */
scatter y1 y2 x, ms(0 S i)        /* extra options are silently ignored */
```

To force all y-variables to use the same option value, use period as placeholder:
```stata
scatter y1 y2 x, ms(. 0) c(. 1)  /* y1: default msymbol; y2: msymbol(0) */
```

---

## 2. Overlay Syntax

### Two equivalent notations

```stata
/* Notation A: parentheses (recommended for complex graphs) */
twoway (scatter y x, ms(O)) (lfit y x, lcolor(red))

/* Notation B: || separator */
twoway scatter y x, ms(O) || lfit y x, lcolor(red)
```

Both are fully equivalent and can be mixed:
```stata
twoway (scatter y1 x) (lfit y1 x) || scatter y2 x
```

### Option scope

**Plot-level options** (after the comma of a specific plottype) affect only that plottype:
```stata
twoway scatter y x, ms(O) || lfit y x, lcolor(red)
/*                  ↑ affects scatter only  ↑ affects lfit only */
```

**twoway_options** (affecting the entire graph) can be attached to any plottype — all produce the same result:
```stata
twoway scatter y x || lfit y x, by(foreign)    /* by() on lfit */
twoway scatter y x, by(foreign) || lfit y x    /* by() on scatter */
twoway (scatter y x) (lfit y x), by(foreign)   /* by() on twoway */
/* All three are identical */
```

**Common trap**: plot-level options placed after the outermost `twoway` comma are interpreted as twoway_options and may cause errors or be silently ignored.

### if / in scope

Each plottype can have its own `if`/`in` condition, independent of others:
```stata
scatter y x if foreign==1 || lfit y x           /* scatter: foreign cars only; lfit: all */
scatter y x if foreign==1 || lfit y x if foreign==1  /* both filtered */
```

### Line continuation

```stata
twoway (scatter y x, ms(O) mcolor(blue))  ///
       (lfit y x, lcolor(red))            ///
       , title("My Graph") scheme(s2color)
```

---

## 3. Dual Y-Axis

> **Full control in `axes.md`**. This section records only the overlay-level binding syntax.

Assign each sub-plot to a y-axis using `yaxis(#)`:
```stata
twoway line y1 x, yaxis(1) || line y2 x, yaxis(2)
```

---

## 4. Plottype Catalog (43 Types)

### Tier 1: High-Frequency Core (full syntax)

| Plottype | Syntax summary | Description |
|----------|---------------|-------------|
| **scatter** | `scatter y1 [y2...] x [, marker_options connect_options colorvar_options jitter_options]` | Scatterplot; mother of all twoway plottypes |
| **line** | `line y1 [y2...] x [, connect_options colorvar_options]` | Line plot; = scatter with msymbol(none) connect(l) |
| **connected** | `connected y1 [y2...] x [, scatter_options]` | Connected points; = scatter with connect(l) |
| **area** | `area y x [, cmissing(y\|n) base(#) sort]` | Area plot, fills between y and baseline |
| **bar** | `bar y x [, base(#) barwidth(#) vertical\|horizontal]` | Bar chart (twoway version, not graph bar) |
| **spike** | `spike y x [, base(#) vertical\|horizontal line_options]` | Spike plot: vertical lines from (y,x) to baseline |
| **dropline** | `dropline y x [, base(#) vertical\|horizontal]` | Dropped-line plot: spike + endpoint marker |
| **lfit** | `lfit y x [, range(# #) n(#) atobs estopts() predopts() cline_options]` | Linear prediction line |
| **lfitci** | `lfitci y x [, stdp\|stdf\|stdr level(#) nofit fitplot(type) ciplot(type)]` | Linear prediction + CI; default CI is rarea |
| **tsline** | `tsline varlist [, scatter_options]` (requires `tsset`) | Time-series line plot |

#### scatter option groups

```stata
scatter y x [,
    /* Marker appearance */
    msymbol(symbolstylelist)    /* shape: O D T S + X i none ... */
    mcolor(colorstylelist)      /* color (with opacity: red%50) */
    msize(markersizestylelist)  /* size: tiny small medium large huge vhuge */
    mfcolor(colorstylelist)     /* fill color */
    mlcolor(colorstylelist)     /* outline color */
    mlwidth(linewidthstylelist) /* outline thickness */
    mstyle(markerstylelist)     /* composite marker style */
    pstyle(pstylelist)          /* composite plot style: p1 p2 ... p15 */

    /* Marker labels */
    mlabel(varlist)             /* label variable(s) */
    mlabposition(clockposlist)  /* position: 0=center 3=right 6=below 9=left 12=above */
    mlabvposition(varname)      /* position from variable */
    mlabsize(textsizestylelist)
    mlabcolor(colorstylelist)
    mlabformat(%fmtlist)

    /* Color by variable */
    colorvar(varname)           /* color points by numeric variable */
    colordiscrete               /* treat colorvar as discrete */
    colorlevels(#)              /* number of color levels */

    /* Connecting points */
    connect(connectstylelist)   /* l=straight J=stairstep etc. */
    sort[(varlist)]             /* sort before connecting */
    cmissing({y|n}...)          /* break line at missing? default y (no break) */
    lpattern(linepatternstylelist)
    lwidth(linewidthstylelist)
    lcolor(colorstylelist)

    /* Jitter (for overplotting) */
    jitter(#)                   /* noise as % of plot area; jitter(5) or jitter(6) common */
    jitterseed(#)               /* random seed for reproducibility */

    /* Axis binding */
    yaxis(# [#...])             /* which y-axis to use */
    xaxis(# [#...])             /* which x-axis to use */
]
```

**pstyle() composite style**: Stata automatically assigns pstyle(p1) to the first sub-plot, p2 to the second, etc. This determines the default color, marker symbol, and line pattern together. To force two sub-plots to share identical styling:
```stata
scatter y1 x || scatter y2 x, pstyle(p1)  /* y2 looks identical to y1 */
```

#### line and scatter relationship

```stata
line y x   /* equivalent to */
scatter y x, msymbol(none) connect(l)
```

- You can use `scatter` in place of `line`, but **not** `line` in place of `scatter`
- `line` accepts all scatter options, but `msymbol()` is ignored (markers are never shown)
- ⚠️ **Caution**: always use `sort` option unless data is already sorted by x — unsorted line plots look like random scribbles

#### lfit / lfitci key options

```stata
lfit y x, range(0 10)      /* specify x range for prediction */
lfit y x, atobs            /* predict only at observed x values */

lfitci y x                         /* default CI: rarea (shaded band) */
lfitci y x, ciplot(rline)          /* change CI to two lines */
lfitci y x, stdp                   /* CI of the mean prediction (default) */
lfitci y x, stdf                   /* CI of individual forecast (wider) */
lfitci y x, level(90)              /* 90% CI */
lfitci y x, nofit                  /* CI only, no prediction line */
lfitci y x, stdf || scatter y x    /* overlay CI first, then scatter on top */
```

⚠️ **Caution**: do not use `lfit` or `qfit` with `yscale(log)` or `xscale(log)` — the prediction is in the original scale, not the log scale, so the resulting curve is not a straight line/parabola as expected.

#### tsline notes

```stata
tsline y1 [y2 ...] [, scatter_options]     /* time-series line, time var set by tsset */
tsrline y1 y2 [, rline_options]            /* time-series range band between two vars */
tsrline y1 y2, recast(rarea)               /* change to filled area */
```

- `tsline`/`tsrline` use `stcolor_alt` scheme by default; `twoway tsline` uses `stcolor`

---

### Tier 2: Moderate Frequency (syntax summary)

| Plottype | Syntax | Description | Key options |
|----------|--------|-------------|-------------|
| **histogram** | `histogram varname [, discrete density\|fraction\|frequency\|percent bins(#) width(#)]` | Histogram (as twoway plottype) | Use standalone `histogram` with `addplot()` for overlaying — `graph twoway histogram` lacks `addplot()` |
| **kdensity** | `kdensity varname [, kernel(epanechnikov\|gaussian\|...) bwidth(#) n(#) area(#)]` | Kernel density estimate | `area(1)` scales area to 1; match y-axis units when overlaying on histogram |
| **lowess** | `lowess y x [, bwidth(#) mean noweight logit adjust]` | Local linear smooth (lowess) | Default bandwidth 0.8; `mean` for running-mean smoothing |
| **lpoly** | `lpoly y x [weight] [, kernel() bwidth(#) degree(#) n(#)]` | Local polynomial regression | Default degree(0) = local mean; more flexible than lowess |
| **lpolyci** | `lpolyci y x [, stdp level(#) nofit fitplot() ciplot()]` | Local polynomial + CI | Same structure as lfitci |
| **qfit** | `qfit y x [, range(##) n(#) atobs estopts()]` | Quadratic prediction | ⚠️ Same log-scale caution as lfit |
| **qfitci** | `qfitci y x [, stdp\|stdf level(#) ciplot()]` | Quadratic prediction + CI | Default CI: rarea; `ciplot(rline)` for lines |
| **fpfit** | `fpfit y x [, degree(#) estopts()]` | Fractional polynomial fit | Based on `fracpoly` command |
| **fpfitci** | `fpfitci y x [, level(#) ciplot()]` | Fractional polynomial + CI | Same structure as lfitci |
| **dot** | `dot y x [, dstyle() dsymbol() dotextend horizontal]` | Dot chart (not graph dot) | Useful for discrete x variables |
| **function** | `function y=f(x), range(a b) [n(#)]` | Mathematical function plot | No data required; `range(0 10)` or `range(xvar)` |

---

### Tier 3: Low-Frequency Catalog (one-line descriptions)

#### Range plots (r* family) → see `ci-bands.md` for full syntax

| Plottype | Description |
|----------|-------------|
| `rarea` | Filled band between two y-variables (standard CI shading) |
| `rcap` | Capped spikes / I-beams (classic error bars) |
| `rcapsym` | Capped spikes with marker symbols at ends |
| `rspike` | Spikes without caps |
| `rbar` | Range bars (filled rectangles between two y-values) |
| `rline` | Two continuous lines at upper and lower bounds |
| `rconnected` | Two connected lines with markers |
| `rscatter` | Markers at both bounds, no connecting lines |
| `rpcap` | Range plot with capped spikes at both bounds and a central point |

#### Paired-coordinate plots (pc* family)

| Plottype | Description |
|----------|-------------|
| `pcspike` | Line connecting (y1,x1) to (y2,x2) |
| `pcarrow` | Arrow from point 1 to point 2 |
| `pcbarrow` | Double-headed arrow |
| `pccapsym` | Capped spike connecting two (y,x) pairs |
| `pcscatter` | Markers at both (y1,x1) and (y2,x2) |
| `pci` | Immediate version of pcspike |
| `pcarrowi` | Immediate version of pcarrow |

#### Other

| Plottype | Description |
|----------|-------------|
| `contour` | Contour fill plot: z y x (requires regular grid) |
| `contourline` | Contour lines only (no fill) |
| `heatmap` | Heat map: z y x |
| `mband` | Median band plot |
| `mspline` | Median spline plot |
| `scatteri` | Immediate scatter — add annotated points without data variables |

#### scatteri usage

```stata
/* Add an annotated point to an existing scatter */
scatter y x || scatteri 22 15 "Note this point"
scatter y x || scatteri 22 15 (3) "At 3 o'clock"  /* (3) = clock position for label */

/* Multiple immediate points */
scatter y x || scatteri 41 2040 "VW Diesel"   ///
                         28 3260 "Plymouth"   ///
                         , msymbol(i) legend(order(1 2 3))
/* msymbol(i) hides the immediate markers, showing only labels */
```

---

## 5. twoway_options Complete Reference

These options apply to **all** twoway plottypes and can be placed on any sub-plot (they affect the entire graph):

| Option | Description | Details in |
|--------|-------------|-----------|
| `added_line_options` | Draw reference lines at specified y or x values (yline/xline) | `axes.md` |
| `added_text_options` | Place text at a (y,x) coordinate | `labels-text.md` |
| `axis_options` | Axis labels, ticks, grids, log scales | `axes.md` |
| `title_options` | title(), subtitle(), note(), caption() | `labels-text.md` |
| `legend_options` | Legend content and styling | `legend.md` |
| `scale(#)` | Scale all text, markers, and line widths; `scale(1.2)` = 20% larger | — |
| `region_options` | Plot region border, background color, graph size (ysize/xsize) | `schemes-regions.md` |
| `aspect_option` | Constrain aspect ratio of the plot region | `schemes-regions.md` |
| `scheme(schemename)` | Overall graph style | `schemes-regions.md` |
| `by(varlist [, subopts])` | Draw separate subgraphs for each group | — |
| `nodraw` | Build graph but do not display it | — |
| `name(name [, replace])` | Store graph in memory under a name (for graph combine) | `export-combine.md` |
| `saving(filename [, replace])` | Save graph to .gph file | `export-combine.md` |
| `advanced_options` | See Section 7 below | — |

### by() scope

`by()` placed on any sub-plot applies to the entire graph — all three forms below are identical:
```stata
twoway scatter y x || lfit y x, by(foreign, total row(1))
twoway scatter y x, by(foreign, total row(1)) || lfit y x
twoway (scatter y x) (lfit y x), by(foreign, total row(1))
```

---

## 6. addplot() Option

Appends twoway sub-plots to graphs produced by **non-graph commands** (e.g., `histogram`, `sts graph`):

```stata
command ..., addplot(plot [|| plot ...] [, below])
```

- `below` draws the added plots **behind** the command's own plots (default is on top)
- ⚠️ `graph twoway histogram` does **not** support `addplot()`; the standalone `histogram` command does

```stata
/* Overlay a normal curve on a histogram */
histogram mpg, normal            /* compact form */
histogram mpg, addplot(function normalden(x, `r(mean)', `r(sd)'), range(mpg))  /* manual equivalent */

/* Overlay a fitted line on a survival curve */
sts graph, addplot(line S_hat _t, sort lpattern(dash))
```

---

## 7. advanced_options Key Items

| Option | Purpose |
|--------|---------|
| `recast(newplottype)` | Re-interpret the current plottype as another plottype |
| `pcycle(#)` | Number of sub-plots before pstyle cycles back to p1; default 15 |
| `yvarlabel("str1" "str2" ...)` | Override y-axis title strings (does not change variable label) |
| `xvarlabel("str")` | Override x-axis title string |
| `yvarformat(%fmt ...)` | Override display format for y variable(s) |
| `yoverhangs` / `xoverhangs` | Adjust margins to prevent long axis labels from clipping |

### recast() details

`recast()` changes how a derived plottype renders its output:
```stata
/* Change lfitci's default CI from rarea to rspike */
lfitci y x, ciplot(rspike)           /* preferred: use ciplot() option */
lfitci y x, recast(line) ciplot(rspike)  /* explicit recast */

/* Plot lfit residuals as a scatterplot instead of a line */
twoway lfit mpg weight, pred(resid) recast(scatter)
```

Recast is valid **within** a family group only:
- **scatter group**: scatter / line / connected / bar / area / spike / dropline / dot
- **range group**: rarea / rbar / rspike / rcap / rcapsym / rline / rconnected / rscatter / rpcap
- **pc group**: pcspike / pccapsym / pcarrow / pcbarrow / pcscatter
- Cross-family recast is not supported

---

## 8. Common Pitfalls

| Pitfall | Explanation | Fix |
|---------|-------------|-----|
| **y x order** | twoway convention: y before x (opposite of `regress y = b*x`) | `scatter y x`, not `scatter x y` |
| **Missing sort** | `line`/`connected` without `sort` on unsorted data → jagged mess | Add `sort` option, or `sort xvar` beforehand |
| **Log scale + fit** | `lfit`/`qfit` + `yscale(log)` or `xscale(log)` → wrong shape | Regress on log-transformed variables, then plot manually |
| **Option scope confusion** | Options after `||` belong to the preceding plottype, not the whole graph | Place twoway_options at the very end, or use `()` notation to be explicit |
| **if/in not inherited** | Each sub-plot's `if` is independent — not passed to other sub-plots | Write `if` on each sub-plot that needs it |
| **Multiple y option order** | `scatter y1 y2 x, ms(O i)`: O→y1, i→y2 — strictly positional | Match option list length to number of y variables |
| **pstyle across plottypes** | `scatter y1 x \|\| line y2 x` auto-assigns p1/p2 — different colors | Use `pstyle(p1)` on y2's plot if you want matching style |
| **cmissing default** | `cmissing(y)` by default — line *does not* break at missing values | Use `cmissing(n)` to break at missing observations |
| **Overlay draw order** | Later sub-plots draw on top of earlier ones | Put background elements (CI bands, reference lines) first; foreground (scatter) last |
| **by() + total** | `by(..., total)` requires `sort` to have been specified | `scatter y x, sort || ..., by(g, total)` |

---

## 9. Quick Option Index

| Need | Option(s) | Reference file |
|------|-----------|---------------|
| Marker shape / size / color | `msymbol()` `msize()` `mcolor()` | `markers.md` |
| Line style / width / color | `lpattern()` `lwidth()` `lcolor()` | `lines.md` |
| Color transparency | `mcolor(red%50)` `fcolor(navy%30)` | `colors.md` |
| Axis labels | `xlabel()` `ylabel()` | `axes.md` |
| Reference lines | `yline(#)` `xline(#)` | `axes.md` |
| Dual y-axis | `yaxis(1)` `yaxis(2)` + `ylabel(, axis(2))` | `axes.md` |
| Legend | `legend(off)` `legend(label(1 "..."))` | `legend.md` |
| Title / subtitle / note | `title()` `subtitle()` `note()` | `labels-text.md` |
| Graph style | `scheme()` | `schemes-regions.md` |
| Export / combine | `graph export` `graph combine` | `export-combine.md` |
| CI / range bands | `rcap` `rarea` `rspike` | `ci-bands.md` |
