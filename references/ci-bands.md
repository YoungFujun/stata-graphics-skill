# ci-bands.md — Confidence Intervals and Error Bands in Stata

Source: Stata 19 Graphics Reference Manual [G-2] and [G-3]; Session B of stata-graphics-skill.

---

## 1. Type Comparison and Selection Guide

All CI/range commands share the same argument order:

```stata
twoway <plottype> y1var y2var xvar [, options]
```

`y1var` and `y2var` are the two bounds (upper/lower, in either order — Stata takes min/max internally). `xvar` is the x-axis variable.

| Command | Visual form | Best for | Notes |
|---------|------------|----------|-------|
| `rcap` | I-beams (spikes + flat end caps) | Event study, coefficient plots | Economics standard; clean and legible |
| `rspike` | Plain spikes (no caps) | Dense plots, minimalist style | Same as rcap but no horizontal end caps |
| `rarea` | Filled shaded band | Continuous fit CI, smooth curves | Requires `sort`; draw before scatter |
| `rcapsym` | Spikes + marker symbols at ends | Rarely used for CI | More suited to labeling than CI display |
| `rbar` | Bars | Not recommended for CI | Visually heavy; suited to range bars, not CI |

**Quick decision rule:**
- Discrete x (event time, coefficient index) → `rcap` or `rspike`
- Continuous x (time series, regression fit) → `rarea`
- Want to label individual endpoints → `rcapsym`

---

## 2. Syntax Reference

### 2.1 `twoway rcap` — Capped spikes (I-beams)

```stata
twoway rcap y1var y2var xvar [if] [in] [, options]
```

| Option | Description |
|--------|-------------|
| `msize(markersizestyle)` | Width of the flat end cap; uses `msymbol()` size values (e.g., `small`, `medium`, `large`, `*1.5`) |
| `vertical` | Vertical spikes; **default** |
| `horizontal` | Horizontal spikes (swap x/y roles) |
| `lcolor(colorstyle)` | Color of spikes and caps |
| `lwidth(linewidthstyle)` | Thickness of spikes and caps |
| `lpattern(linepatternstyle)` | Pattern of spike lines |
| `colorvar_options` | Color spikes by values of a variable |

All options are *rightmost* (can be repeated; last wins), except `vertical`/`horizontal` which are *unique*.

**Quick start examples:**
```stata
twoway rcap ci_hi ci_lo t                      // basic
twoway rcap ci_hi ci_lo t, msize(small)        // narrower caps
twoway rcap ci_hi ci_lo t, msize(small) lcolor(orange)
twoway rcap ci_hi ci_lo t, horizontal          // horizontal spikes
```

---

### 2.2 `twoway rspike` — Plain spikes (no caps)

```stata
twoway rspike y1var y2var xvar [if] [in] [, options]
```

| Option | Description |
|--------|-------------|
| `lpattern(linepatternstyle)` | spike line pattern (solid, dash, dot, …) |
| `lwidth(linewidthstyle)` | thickness of spike line |
| `lcolor(colorstyle)` | color and opacity of spike line |
| `lstyle(linestyle)` | overall spike line style |
| `vertical` | vertical spikes; **default** |
| `horizontal` | horizontal spikes |

All options are *rightmost*.

**Quick start examples:**
```stata
twoway rspike ci_hi ci_lo t
twoway rspike ci_hi ci_lo t, lcolor(maroon)
twoway rspike ci_hi ci_lo t || line coef t, sort   // overlaid line
```

---

### 2.3 `twoway rarea` — Filled shaded band

```stata
twoway rarea y1var y2var xvar [if] [in] [, options]
```

| Option | Description |
|--------|-------------|
| `sort` | Sort data by xvar before plotting; **always include** |
| `fcolor(colorstyle)` | Fill color and opacity (e.g., `gs14`, `navy%30`) |
| `fintensity(intensitystyle)` | Fill intensity (e.g., `fintensity(30)` for 30%) |
| `color(colorstyle)` | Both fill and outline color |
| `lcolor(colorstyle)` | Outline color |
| `lwidth(linewidthstyle)` | Outline thickness |
| `lpattern(linepatternstyle)` | Outline pattern |
| `vertical` | vertical area plot; **default** |
| `horizontal` | horizontal area plot |
| `cmissing(y\|n)` | How to handle missing values; default `cmissing(y)` (ignored); use `cmissing(n)` to create gaps |

All options are *unique*.

**Quick start examples:**
```stata
twoway rarea ci_lo ci_hi t, sort                               // default fill
twoway rarea ci_lo ci_hi t, sort fcolor(gs14) lcolor(gs14)    // light gray, no border
twoway rarea ci_lo ci_hi t, sort fcolor(navy) fintensity(30) lcolor(navy)
```

**Critical caution:** Data must be in order of `xvar`, or specify `sort` option. Unsorted data produces a jagged, incorrect fill.

---

### 2.4 `twoway rcapsym` — Spikes with marker symbols

```stata
twoway rcapsym y1var y2var xvar [if] [in] [, options]
```

| Option | Description |
|--------|-------------|
| `vertical` / `horizontal` | orientation |
| `marker_options` | shape, size, color of end symbols (same marker used on both ends) |
| `msymbol(symbolstyle)` | symbol type (e.g., `diamond_hollow`, `circle`) |
| `mlabel(varname)` | label endpoints (limited use — same label on both ends) |
| `line_options` | spike line look |

**Note:** `rcapsym` is seldom the right choice for CI visualization; `rcap` is almost always preferable.

---

## 3. Options Reference

### 3.1 `area_options` — For `rarea` fill appearance

Applies to `twoway area` and `twoway rarea`.

| Option | Description |
|--------|-------------|
| `color(colorstyle)` | Outline and fill color (both at once) |
| `fcolor(colorstyle)` | Fill color and opacity |
| `fintensity(intensitystyle)` | Fill intensity (0–100) |
| `lcolor(colorstyle)` | Outline color |
| `lwidth(linewidthstyle)` | Outline thickness |
| `lpattern(linepatternstyle)` | Outline pattern |
| `lalign(linealignmentstyle)` | Outline alignment |
| `lstyle(linestyle)` | Overall outline style |
| `astyle(areastyle)` | Overall area style (starting point for all above) |
| `pstyle(pstyle)` | Overall plot style |
| `recast(newplottype)` | Advanced: recast plot type |

All `area_options` are *merged-implicit* (options combine with defaults rather than overriding them).

### 3.2 `rcap_options` — For `rcap` cap appearance

| Option | Description |
|--------|-------------|
| `line_options` | Spike and cap line look (lpattern, lwidth, lcolor, lstyle) |
| `msize(markersizestyle)` | Cap width; borrows scatter's marker size system |
| `recast(newplottype)` | Advanced |

All `rcap_options` are *rightmost*.

### 3.3 `rspike_options` — For `rspike` spike appearance

| Option | Description |
|--------|-------------|
| `lpattern(linepatternstyle)` | Spike line pattern |
| `lwidth(linewidthstyle)` | Spike line thickness |
| `lcolor(colorstyle)` | Spike color and opacity |
| `lstyle(linestyle)` | Overall spike line style |
| `pstyle(pstyle)` | Overall plot style |
| `recast(newplottype)` | Advanced |

All `rspike_options` are *rightmost*.

### 3.4 `fitarea_options` — For fitted CI area (lfitci, qfitci, etc.)

Used with `twoway lfitci`, `twoway qfitci`, `twoway lpolyci`, etc. (not `rarea`).

| Option | Description |
|--------|-------------|
| `acolor(colorstyle)` | Outline and fill color |
| `fcolor(colorstyle)` | Fill color and opacity |
| `fintensity(intensitystyle)` | Fill intensity |
| `alcolor(colorstyle)` | Outline color |
| `alwidth(linewidthstyle)` | Outline thickness |
| `alpattern(linepatternstyle)` | Outline pattern |
| `astyle(areastyle)` | Overall area style |
| `pstyle(pstyle)` | Overall plot style |

---

## 4. Overlay Ordering Rule

**Always draw CI/range elements FIRST, then scatter points on top.**

```stata
// CORRECT: rarea drawn first, scatter on top
twoway rarea ci_lo ci_hi t, sort fcolor(gs14) lcolor(gs14) ///
    || scatter coef t, msymbol(circle) mcolor(navy)

// WRONG: scatter drawn first, then covered by rarea shading
twoway scatter coef t ///
    || rarea ci_lo ci_hi t, sort    // shading covers scatter points
```

This applies equally to `rcap` + `scatter` combinations:
```stata
// rcap drawn first, scatter (coefficient dot) on top
twoway rcap ci_hi ci_lo t ///
    || scatter coef t
```

---

## 5. Standard CI Construction (Manual)

When CI bounds are not already stored as variables, generate them from regression stored results:

```stata
* After regress / reghdfe:
predict hat, xb
predict s, stdp              // SE of prediction
generate ci_lo = hat - 1.96*s
generate ci_hi = hat + 1.96*s

twoway rarea ci_lo ci_hi xvar, sort fcolor(gs14) lcolor(gs14) ///
    || scatter yvar xvar
```

For `margins` output, use `marginsplot` directly or see `estimation-to-graph.md` for manual extraction.

---

## 6. Event Study Template

Full event study graph using `rcap` + `scatter`:

```stata
* Assumes: coef_b = point estimate, ci_lo = lower 95% CI, ci_hi = upper 95% CI
* stored in a dataset indexed by event_time

twoway ///
    (rcap ci_hi ci_lo event_time, lcolor(gs8) msize(small)) ///
    (scatter coef_b event_time, ///
        msymbol(circle) mcolor(navy) msize(small)) ///
    , ///
    yline(0, lcolor(black) lpattern(solid) lwidth(thin)) ///
    xline(-0.5, lcolor(gs8) lpattern(dash) lwidth(thin)) ///
    xlabel(-5(1)5) ///
    xtitle("Event time (years)") ///
    ytitle("Coefficient") ///
    legend(off) ///
    scheme(s2mono)
```

**With `rarea` for shaded CI band:**

```stata
twoway ///
    (rarea ci_lo ci_hi event_time, sort fcolor(gs14) lcolor(gs14)) ///
    (line coef_b event_time, sort lcolor(navy) lwidth(medthick)) ///
    , ///
    yline(0, lcolor(black) lpattern(solid)) ///
    xline(-0.5, lcolor(gs8) lpattern(dash)) ///
    xlabel(-5(1)5) ///
    xtitle("Event time") ytitle("Coefficient") ///
    legend(off) scheme(s2mono)
```

See `graph-templates.md` for additional event study and coefficient plot templates (coefplot version, multi-model comparison, csdid_plot).

---

## 7. Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Forgot `sort` in `rarea` | Jagged / modern-art fill | Add `, sort` option |
| Wrong overlay order | Scatter points hidden under shading | Draw `rarea`/`rcap` first, `scatter` last |
| `rcap` argument order confusion | Extra-long or zero-length spikes | Both `y1var y2var` are bounds; Stata takes min/max internally — order doesn't matter |
| `rspike` mistaken for `rcap` | No end caps visible | Use `rcap` if you need visible caps |
| `fintensity` vs `%` opacity | Unexpected color | Use `fcolor(navy%30)` (Stata 15+) or `fintensity(30)` for fill intensity |
| `rarea` CI crossed | Inverted shading when CI bounds cross | Normal behavior; pre-check data for CI validity |
