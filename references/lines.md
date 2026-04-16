# Lines in Stata Graphics

Sources: [G-3] line_options, connect_options, [G-4] linepatternstyle, linewidthstyle, linestyle, connectstyle — Stata 19 Graphics Reference Manual

---

## line_options — Full Syntax

```stata
lcolor(colorstyle)          // line color
lwidth(linewidthstyle)      // line thickness
lpattern(linepatternstyle)  // dash/dot pattern
lstyle(linestyle)           // preset line style (overrides individual options)
pstyle(pstyle)              // preset plot style
recast(newplottype)         // change plot type after rendering
```

**Option precedence:** `lstyle()` sets color + width + pattern as a bundle;
individual options (`lcolor`, `lwidth`, `lpattern`) override specific attributes.

---

## Line Pattern (linepatternstyle)

Specified via `lpattern()`.

### Named patterns

| linepatternstyle  | Description                     |
|-------------------|---------------------------------|
| `solid`           | ────────── (default)            |
| `dash`            | ─ ─ ─ ─ ─                      |
| `dot`             | ··········                      |
| `dash_dot`        | ─·─·─·─·                       |
| `shortdash`       | shorter dashes than `dash`      |
| `shortdash_dot`   | short dash + dot                |
| `longdash`        | longer dashes than `dash`       |
| `longdash_dot`    | long dash + dot                 |
| `blank`           | invisible line                  |

### Formula syntax

Custom patterns are specified as a quoted string using these characters:

| Character | Meaning            |
|-----------|--------------------|
| `l`       | solid segment      |
| `_`       | space (gap)        |
| `-`       | dash               |
| `.`       | dot                |
| `#`       | numeric gap width  |

```stata
lpattern("l_l_")          // solid-gap-solid-gap (equivalent to dash)
lpattern("-_._")          // dash-gap-dot-gap
lpattern("l__l__")        // solid-long gap-solid-long gap
```

---

## Line Width (linewidthstyle)

Specified via `lwidth()`.

### Named widths (thinnest to thickest)

```
none  vvvthin  vvthin  vthin  thin  medthin  medium  medthick  thick  vthick  vvthick  vvvthick
```

`none` = zero-width line (invisible but technically present — use `lpattern(blank)` for no line).

Default width in most schemes is `medium` or `medthin`.

### Numeric and unit widths

Follows the general `size` syntax:

| Format | Meaning                          | Example            |
|--------|----------------------------------|--------------------|
| `#`    | scheme-dependent relative units  | `lwidth(1)`        |
| `#pt`  | printer points                   | `lwidth(1pt)`      |
| `#in`  | inches                           | `lwidth(.01in)`    |
| `#cm`  | centimeters                      | `lwidth(.03cm)`    |
| `*#`   | multiply current width by #      | `lwidth(*1.5)`     |

```stata
lwidth(thin)     // named
lwidth(medthick) // named (good for emphasis)
lwidth(1pt)      // absolute
lwidth(*2)       // twice the current width
```

---

## Line Style Presets (linestyle)

`lstyle()` selects a predefined combination of color + width + pattern.

### Semantic styles

| linestyle      | Description                              |
|----------------|------------------------------------------|
| `foreground`   | default foreground line                  |
| `grid`         | grid line style                          |
| `minor_grid`   | minor grid line                          |
| `major_grid`   | major grid line                          |
| `refline`      | reference line (xline, yline)            |
| `yxline`       | y=x reference line                       |
| `none`         | no line                                  |

### Plot line families (scheme-determined)

| Family           | Used by                                  |
|------------------|------------------------------------------|
| `p1`–`p15`       | first–fifteenth twoway plot line         |
| `p1bar`–`p15bar` | bar outlines                             |
| `p1box`–`p15box` | box outlines                             |
| `p1area`–`p15area`| area plot outlines                      |
| `p1solid`–`p15solid`| solid-fill outlines                  |
| `p1mark`–`p15mark` | marker outlines                       |
| `p1other`–`p15other`| other plot type lines                |

Attributes can be overridden after specifying `lstyle()`:

```stata
twoway line y x, lstyle(p1) lcolor(red)   // p1 preset, override color
```

---

## Connect Options (connect_options)

These control *how* data points in a line or connected plot are joined.

```stata
connect(connectstyle)     // connection method
cmissing(y|n [y|n ...])  // y = skip over missing (connect gap); n = break line at missing
sort                      // sort by x before connecting
sort(varlist)             // sort by varlist before connecting
```

### connectstyle values

| connectstyle | Synonym | Description                                          |
|--------------|---------|------------------------------------------------------|
| `none`       | `i`     | do not connect (markers only)                        |
| `direct`     | `l`     | straight line between consecutive observations       |
| `ascending`  | `L`     | connect only when x is nondecreasing (skips drops)   |
| `stairstep`  | `J`     | horizontal then vertical step                        |
| `stepstair`  | —       | vertical then horizontal step                        |

**Important:** For `line` and `connected`, data must be sorted by x before connecting,
otherwise lines cross back. Use `sort` option or pre-sort the dataset.

```stata
twoway line y x, sort                    // sort before drawing
twoway connected y x, sort connect(J)   // stairstep
twoway line y x, connect(l) cmissing(n) // break at missing values
```

---

## Common Line Option Patterns

### Publication-quality reference line
```stata
yline(0, lcolor(black) lwidth(thin) lpattern(dash))
xline(0, lcolor(gs8) lwidth(vthin) lpattern(solid))
```

### Confidence band with line
```stata
twoway ///
  (rarea ub lb x, fcolor(navy%15) lwidth(none)) ///
  (line b x,      lcolor(navy) lwidth(medthick))
```

### Multiple series with distinct line styles
```stata
twoway ///
  (line y1 x, lcolor(navy)   lpattern(solid)  lwidth(medthick)) ///
  (line y2 x, lcolor(maroon) lpattern(dash)   lwidth(medthick)) ///
  (line y3 x, lcolor(forest_green) lpattern(dot) lwidth(medthick))
```

### Event study trend line
```stata
twoway connected b t, ///
  sort                        /// ensure x-order
  lcolor(navy) lwidth(medium) ///
  msymbol(O) msize(small) mcolor(navy)
```

### Suppress line, show markers only
```stata
twoway scatter y x            // scatter has no line by default
twoway connected y x, connect(i)  // explicitly: no connection
```

### Invisible line for plot structure (combine with rcap)
```stata
twoway ///
  (rcap ub lb t,  lcolor(navy) lwidth(medthick)) ///
  (scatter b t,   msymbol(O) msize(small) mcolor(navy))
// rcap draws CI caps; scatter draws point estimates — no explicit "line" needed
```

---

## Line Color and Opacity

```stata
lcolor(navy)          // named color
lcolor(gs8)           // gray scale 0–16 (0=black, 16=white)
lcolor("51 102 153")  // RGB triple
lcolor(navy%50)       // 50% opacity
lcolor(none)          // transparent line
```

---

## Multiple Plots: linepatternstylelist / linewidthstylelist

When multiple plots share an option, provide a space-separated list:

```stata
twoway line y1 y2 y3 x, lpattern(solid dash dot)
twoway line y1 y2 x,    lwidth(medthick thin)
twoway line y1 y2 x,    lcolor(navy maroon)
```

Stylelist shorthands:
- `.` — default for this element
- `=` — repeat previous element  
- `..` — repeat previous to end

```stata
lpattern(solid . .)    // solid, then default for rest
lwidth(medthick = =)   // medthick for all three
```
