# labels-text.md — Titles, Annotations, and Marker Labels in Stata

Source: Stata 19 Graphics Reference Manual [G-3] `added_text_options` (pp.451-457),
`marker_label_options` (pp.595-604), `textbox_options` (pp.648-656),
`title_options` (pp.658-667); [G-4] `text` (pp.765-775); Mitchell (2022)
§8.2, §8.10-§8.12, §9.1.

---

## 1. Overview: Four Text Systems

Stata graphics use four related but distinct text systems:

| System | Main syntax | Best use |
|--------|-------------|----------|
| Graph titles | `title()`, `subtitle()`, `note()`, `caption()` | Explain the graph as a whole |
| Added annotations | `text()` / `ttext()` | Label an outlier, policy date, regime, or special region inside the plot |
| Marker labels | `mlabel()` + `mlab*` | Label points observation-by-observation |
| Textbox rendering | `textbox_options` | Control size, color, margin, border, justification, width, etc. |

**Division of labor:**
- `title()` / `subtitle()` / `note()` / `caption()` belong here.
- `xtitle()` / `ytitle()` are covered in [axes.md](./axes.md).
- Legend text belongs in `legend.md` (Session E2).

**Practical decision rule:**
- Need one or a few custom labels in known coordinates -> use `text()`
- Need labels tied to many observations -> use `mlabel()`
- Need graph-level explanation/source note -> use `title()` / `subtitle()` / `note()` / `caption()`
- Need a box, fill, justification, margin, or custom width -> add `textbox_options`

---

## 2. title_options — Graph-Level Titles, Notes, and Captions

### 2.1 Syntax

```stata
title("string" ["string" ...] [, suboptions textbox_options])
subtitle("string" ["string" ...] [, suboptions textbox_options])
note("string" ["string" ...] [, suboptions textbox_options])
caption("string" ["string" ...] [, suboptions textbox_options])
```

Less commonly, `graph twoway` also allows:

```stata
t1title() t2title() b1title() b2title()
l1title() l2title() r1title() r2title()
```

These are rarely needed. For axis titles, prefer `xtitle()` / `ytitle()`; see [axes.md](./axes.md).
Also note: `position()` is **not** allowed with `t1title()` / `t2title()` / `b1title()` / `b2title()` /
`l1title()` / `l2title()` / `r1title()` / `r2title()`.

### 2.2 Default roles

| Option | Default location | Typical use |
|--------|------------------|-------------|
| `title()` | Top center, largest text | Main graph title |
| `subtitle()` | Near the title, slightly smaller | Sample / period / subgroup description |
| `note()` | Bottom-left, small text | Source note, caveat, estimation detail |
| `caption()` | Below note, slightly larger than note | Short explanation or figure caption text |

If `title()` feels too large, using `subtitle()` alone is often cleaner.

### 2.3 Suboptions

| Suboption | Description |
|-----------|-------------|
| `prefix` | Add text as a separate line before an existing title of the same type |
| `suffix` | Add text as a separate line after an existing title of the same type |
| `position(clockposstyle)` | Place title by clock position around the plot region |
| `ring(ringposstyle)` | Distance from plot region; `ring(0)` means inside plot region |
| `span` | Span the full graph width/height rather than the plot region only |
| `textbox_options` | Box, size, color, justification, width, margin, etc. |

`title()`, `subtitle()`, `note()`, and `caption()` are **merged-explicit**. Repeated options are combined deliberately rather than last-one-wins.

### 2.4 Prefix and suffix

`prefix` and `suffix` add separate lines before or after an existing title of the same type.

```stata
// Existing title + appended line
scatter y x, title("Main title") title("Second line", suffix)

// Existing note + prepended line
scatter y x, note("Source: Authors' calculations") ///
             note("Sample restricted to balanced panel", prefix)
```

### 2.5 Multi-line titles and quotes

```stata
// Multi-line title: one quoted string per line
scatter y x, title("Main title" "Second line")

// Subtitle only
scatter y x, subtitle("Smaller main title")

// Quotes inside the string: use compound quotes
scatter y x, title(`"A "quoted" title"' )
```

### 2.6 Placement with position() and ring()

`position()` follows clock positions around the plot region:

| Value | Meaning |
|-------|---------|
| `12` | Top center |
| `6` | Bottom center |
| `3` | Right side |
| `9` | Left side |
| `1`, `2`, `4`, `5`, `7`, `8`, `10`, `11` | Corner/edge variants |

```stata
// Bottom title
scatter y x, title("My title", position(6))

// Bottom-left title
scatter y x, title("My title", position(7))

// Inside the plot region
scatter y x, title("Inside plot", position(1) ring(0))
```

### 2.7 Spanning

By default, title-related text usually aligns to the **plot region**, not the full graph width.
`span` switches alignment to the full graph area.

```stata
// Standard centered title over plot region
scatter y x, title("Main title")

// Span full graph width
scatter y x, title("Main title", span)

// If title spans, subtitle should usually span too
scatter y x, title("Main title", span) ///
             subtitle("Sample: 2000-2020", span)
```

`span` is especially useful for:
- Wide titles over a graph with a right-side legend
- Notes/captions that should align with the full graph, not just the plot area
- Multi-panel layouts where consistent left edge alignment matters

### 2.8 Examples

```stata
// Standard economics figure title + source note
twoway scatter y x, ///
    title("Event-study estimates") ///
    subtitle("Baseline sample") ///
    note("Source: Authors' calculations")

// Put note inside the plot region
twoway scatter y x, ///
    note("95% confidence intervals omitted for clarity", ///
         ring(0) position(5) box fcolor(gs15) lcolor(gs10))

// Multi-line title with full-width box
twoway scatter y x, ///
    title("Treatment effects by cohort" ///
          "Specification with county and year FE", ///
          box bexpand span justification(left))
```

---

## 3. textbox_options — Rendering Text and Textboxes

`textbox_options` are shared by titles, notes, captions, and `text()` annotations.
They control **how the text box looks**, not where the surrounding graph framework places it.

### 3.1 Syntax

```stata
tstyle(textboxstyle)
orientation(orientationstyle)
size(textsizestyle)
color(colorstyle)
justification(justificationstyle)
alignment(alignmentstyle)
margin(marginstyle)
linegap(size)
width(size)
height(size)

box | nobox
bcolor(colorstyle)
fcolor(colorstyle)
lstyle(linestyle)
lpattern(linepatternstyle)
lwidth(linewidthstyle)
lcolor(colorstyle)
lalign(linealignmentstyle)

bmargin(marginstyle)
bexpand
placement(compassdirstyle)
```

### 3.2 Core text appearance options

| Option | Description |
|--------|-------------|
| `tstyle(textboxstyle)` | Overall textbox style bundle; rarely specified directly |
| `orientation(horizontal\|vertical\|rvertical)` | Reading direction of the textbox |
| `size(textsizestyle)` | Text size |
| `color(colorstyle)` | Text color and opacity |
| `justification(left\|center\|right)` | Horizontal alignment of lines inside a multi-line textbox |
| `alignment(top\|middle\|baseline\|bottom)` | Vertical alignment inside the box |
| `linegap(size)` | Distance between lines |
| `width(size)` / `height(size)` | Override auto-computed box dimensions |

`orientation(vertical)` reads bottom-to-top; `orientation(rvertical)` reads top-to-bottom.

### 3.3 Box and border options

| Option | Description |
|--------|-------------|
| `box` / `nobox` | Draw or suppress the border |
| `bcolor(colorstyle)` | Background + border color together |
| `fcolor(colorstyle)` | Background fill color only |
| `lstyle(linestyle)` | Overall border style |
| `lpattern(linepatternstyle)` | Border pattern |
| `lwidth(linewidthstyle)` | Border thickness |
| `lcolor(colorstyle)` | Border color only |
| `lalign(linealignmentstyle)` | Border inside / outside / center |

### 3.4 Margin logic: `margin()` vs `bmargin()`

There are **two different margins**:

| Option | Meaning |
|--------|---------|
| `margin()` | Padding from text to the border of the textbox |
| `bmargin()` | Margin from the textbox border outward to the containing box |

```stata
// Padding inside the textbox
title("My title", box margin(medsmall))

// Nudge the textbox within its containing area
caption("My caption", ring(0) position(7) bmargin(2 0 2 0))
```

Use `margin()` when the box looks cramped. Use `bmargin()` when the text box needs to be shifted inside the space where it is already being placed.
For rotated textboxes, left/right/top/bottom margins refer to the margins **before rotation**.

### 3.5 Justification is not placement

`justification()` controls how lines align **inside** the textbox:

```stata
title("First line" "Second line", justification(left))
```

This does **not** move the textbox itself. Textbox location is determined by the surrounding command, such as `position()` / `ring()` for titles or `placement()` for `text()`.

### 3.6 Width and height overrides

Stata approximates textbox width. When a box border is visible, auto-width can look wrong:
- text runs outside the box
- too much empty space appears on the right

In those cases, use `width()` as a manual override.

```stata
text(48.5 1923 ///
    "The 1918 influenza pandemic was the worst epidemic" ///
    "known in the US.", ///
    placement(se) box fcolor(gs15) width(85))
```

`width()` usually requires trial and error. `height()` exists too, but is much less often needed.

### 3.7 `bexpand`

`bexpand` expands the textbox in the direction of the text:
- horizontal text -> wider
- vertical text -> longer

Most often used with titles:

```stata
scatter y x, title("This is my" "title", box bexpand)
scatter y x, title("This is my" "title", box bexpand span)
```

### 3.8 `placement()` as override

`placement()` is part of `textbox_options`, but for `text()` you normally use its own placement logic directly; see Section 4.
For titles and other contextual textboxes, `placement()` is mostly an override when default/context placement is not enough.

---

## 4. text() and ttext() — Custom Annotations Inside the Plot

### 4.1 Syntax

```stata
text(y x "text" ["text" ...] [ y x "text" ... ] [, suboptions])
ttext(y t "text" ["text" ...] [ y t "text" ... ] [, suboptions])
```

`text()` and `ttext()` are **merged-implicit** and may be repeated.

Repeated `text()` calls are evaluated separately, so different annotation boxes can use different styles.

Where suboptions are:

| Suboption | Description |
|-----------|-------------|
| `yaxis(#)` / `xaxis(#)` | Which axis scale the coordinates refer to |
| `placement(compassdirstyle)` | Where the textbox sits relative to `(y, x)` |
| `textbox_options` | Box, fill, size, color, justification, width, etc. |

### 4.2 Coordinate rule

`text()` uses **data coordinates**:
- first number = y coordinate
- second number = x coordinate

```stata
// Label the point (x=62, y=47)
scatter y x, text(47 62 "Washington, DC")
```

`ttext()` is the same idea but accepts dates for time-formatted x-axes:

```stata
tsline y, ttext(1035 01apr2001 "Start of Q2")
```

### 4.3 Placement values

| Value | Meaning |
|-------|---------|
| `c` | centered on the point |
| `n` | above |
| `ne` | above-right |
| `e` | right |
| `se` | below-right |
| `s` | below |
| `sw` | below-left |
| `w` | left |
| `nw` | above-left |

```stata
// Text to the right of the point
scatter y x, text(7 11 "My text", placement(e))

// Centered below the point
scatter y x, text(7 11 "My text", placement(s))

// Repeated text() with different styling
scatter y x, ///
    text(7 11 "Point A", placement(e)) ///
    text(9 14 "Point B", placement(sw) box fcolor(gs15))
```

### 4.4 Typical use: annotate a few important points

```stata
twoway qfitci mpg weight, stdf ///
    || scatter mpg weight, msymbol(O) ///
    text(41 2040 "VW Diesel", placement(e)) ///
    text(28 3260 "Plymouth Arrow", placement(e)) ///
    text(35 2050 "Datsun 210 and Subaru", placement(e))
```

Use `text()` when you want to label only selected observations rather than every point.

### 4.5 Explanatory text box inside the graph

```stata
twoway line le year || fpfit le year, ///
    ytitle("Life expectancy, years") ///
    xlabel(1900 1918 1940(20)2000) ///
    title("Life expectancy at birth") ///
    subtitle("US, 1900 to 1999") ///
    note("Source: National Vital Statistics Report, Vol. 50 No. 6") ///
    legend(off) ///
    text(48.5 1923 ///
         "The 1918 influenza pandemic was the worst epidemic" ///
         "known in the US." ///
         "More citizens died than in all combat deaths of the" ///
         "20th century.", ///
         placement(se) box fcolor(gs15) justification(left) ///
         margin(l+4 t+1 b+1) width(85))
```

This is the canonical pattern for an in-plot annotation box.

### 4.6 Multiple axes

When multiple x or y axes exist, specify which scale the coordinates belong to:

```stata
twoway (scatter gnp year, yaxis(1)) ///
       (scatter rate year, yaxis(2)), ///
       text(1200 1980 "Output", yaxis(1)) ///
       text(8 1980 "Rate", yaxis(2))
```

Without `yaxis()` / `xaxis()`, Stata assumes axis 1.

---

## 5. marker_label_options — Labels Attached to Points

### 5.1 Syntax

```stata
mlabel(varname)
mlabstyle(markerlabelstyle)
mlabposition(clockposstyle)
mlabvposition(varname)
mlabgap(size)
mlabangle(anglestyle)
mlabtextstyle(textstyle)
mlabsize(textsizestyle)
mlabcolor(colorstyle)
mlabformat(%fmt)
mlabprefix(text)
mlabsuffix(text)
```

These options are **rightmost**. With `scatter`, style lists are often allowed.
When lists are allowed, `mlabel()` may take a `varlist` rather than a single variable.

### 5.2 Core options

| Option | Description |
|--------|-------------|
| `mlabel(varname)` | Variable supplying label text, usually string |
| `mlabstyle(markerlabelstyle)` | Overall marker-label style bundle |
| `mlabposition(clockposstyle)` | Constant label position for all points |
| `mlabvposition(varname)` | Observation-specific positions, values `0-12` |
| `mlabgap(size)` | Distance from marker to label |
| `mlabangle(anglestyle)` | Text angle |
| `mlabtextstyle(textstyle)` | Overall text style bundle for marker labels |
| `mlabsize(textsizestyle)` | Label size |
| `mlabcolor(colorstyle)` | Label color and opacity |
| `mlabformat(%fmt)` | Numeric format for numeric labels |
| `mlabprefix(text)` / `mlabsuffix(text)` | Add prefix/suffix around each label |

### 5.3 `mlabposition()` clock system

Marker label positions use clock positions:

| Value | Meaning |
|-------|---------|
| `12` | above |
| `3` | right |
| `6` | below |
| `9` | left |
| `1`, `2`, `4`, `5`, `7`, `8`, `10`, `11` | diagonal positions |
| `0` | directly on top of the point |

Default is usually `mlabposition(3)`.

```stata
// Label to the left of every point
scatter y x, mlabel(name) mlabposition(9)

// Put the label on the point itself
scatter y x, mlabel(name) mlabposition(0) msymbol(i)
```

If you use `mlabposition(0)`, hide the marker with `msymbol(i)` unless you intentionally want marker and text to overlap.

### 5.4 `mlabvposition()` for selective conflict resolution

`mlabvposition()` is one of the most important practical tools in this file.
Use it when most labels can share one position, but a few need manual adjustment.

```stata
generate pos = 3
replace pos = 9 if country == "Honduras"

scatter lexp gnppc if region==2, ///
    mlabel(country) ///
    mlabvposition(pos) ///
    xscale(range(35000)) ///
    plotregion(margin(l+9))
```

This is usually better than forcing one `mlabposition()` on every point.

### 5.5 Gap, angle, and text-style control

```stata
// Move labels farther from the marker
scatter y x, mlabel(name) mlabgap(2)

// Rotate labels
scatter y x, mlabel(name) mlabangle(45)

// Use bundle style, then override one attribute
scatter y x, mlabel(name) mlabtextstyle(small) mlabcolor(maroon)
```

As with `mlabstyle()`, `mlabtextstyle()` is a starting point; specific options such as
`mlabsize()` and `mlabcolor()` override parts of it.

### 5.6 Numeric labels

```stata
// Numeric values as labels
scatter y x, mlabel(id)

// Numeric labels with formatting
scatter y x, mlabel(pvalue) mlabformat(%4.2f)

// Prefix/suffix
scatter y x, mlabel(beta) mlabprefix("b = ") mlabsuffix("**")
```

### 5.7 Selective labeling patterns

```stata
// Label only selected observations
scatter y x if inlist(id, 1, 5, 8), mlabel(name)

// Hide marker and use labels as the visual mark
scatter y x, msymbol(i) mlabel(name) mlabposition(0)
```

For dense figures, selective labeling is often superior to labeling every point.

---

## 6. Graph Text and SMCL

All graph text supports Unicode plus many SMCL tags.
This applies to `title()`, `note()`, `caption()`, `text()`, and marker labels.

### 6.1 Common tags

| Tag | Use |
|-----|-----|
| `{bf:text}` | bold |
| `{it:text}` | italics |
| `{sup:text}` / `{superscript:text}` | superscript |
| `{sub:text}` / `{subscript:text}` | subscript |
| `{stSans:text}` | sans-serif font |
| `{stSerif:text}` | serif font |
| `{stMono:text}` | monospace font |
| `{stSymbol:text}` | symbol font |
| `{fontface "Font Name":text}` | system font override |
| `{&Sigma}` | symbol / Greek letter |
| `{&le}` | <= symbol |
| `{c a'}` | accented character via SMCL |

### 6.2 Examples

```stata
// Bold and italics
note("{bf:Source}: {it:Consumer Reports}")

// Superscripts and subscripts
title("{&function}(x)=2e{sup:-2x}")
note("Note: m{sub:1} s{sup:2}")

// Greek letters and symbols
title("{&Sigma} {&mu} {&le} 0{&degree}")

// Serif / sans / mono
title("{stSerif:Serif title}") ///
subtitle("{stSans:Sans subtitle}") ///
note("{stMono:Monospace note}")
```

### 6.3 Font caution

`{fontface ...}` can use system fonts, but portability is weaker:
- another machine may not have the same font
- PostScript / EPS export may fail or substitute fonts

For reproducible academic workflows, prefer the standard Stata font families unless the journal or house style requires otherwise.

---

## 7. Common Traps

### Trap 1 — `justification()` does not move the textbox

```stata
// ❌ Wrong mental model: this moves the title to the left edge
title("My title", justification(left))

// ✅ What it actually does: left-justify text INSIDE the title box
// Use position()/ring()/span to move the title box itself
title("My title", position(11) span)
```

### Trap 2 — `text()` uses data coordinates, not screen coordinates

```stata
// If x/y ranges change, this annotation will move relative to the data
twoway scatter y x, text(47 62 "DC")
```

If you rescale axes, filter observations, or switch to a different sample, `text()` positions may need to be recomputed.

### Trap 3 — `mlabel()` is usually a bad idea for dense scatterplots

```stata
// ❌ Usually unreadable
scatter y x, mlabel(name)

// ✅ Better: label selected points only, or use text() for a few outliers
scatter y x if highlight, mlabel(name)
```

Marker labels work well only when there are few observations or when labels are heavily curated.

This is not just aesthetics; both the official manual and Mitchell repeatedly treat marker labels as a
small-`N` tool.

### Trap 4 — `mlabposition()` and `mlabvposition()` are not interchangeable

```stata
// Constant position for all points
scatter y x, mlabel(name) mlabposition(3)

// Observation-specific positions
scatter y x, mlabel(name) mlabvposition(posvar)
```

If both are specified, `mlabvposition()` takes precedence.

### Trap 5 — `mlabposition(0)` should usually be paired with `msymbol(i)`

```stata
// ❌ Label and marker printed on top of each other
scatter y x, mlabel(name) mlabposition(0)

// ✅ Text replaces marker visually
scatter y x, mlabel(name) mlabposition(0) msymbol(i)
```

### Trap 6 — `width()` is often the fix for ugly boxed annotations

```stata
// If text spills outside the box or leaves too much empty right margin,
// override width manually
text(..., box width(85))
```

This is a Stata textbox width-approximation issue, not necessarily a mistake in your text.

### Trap 7 — `bcolor()` often hides the border

```stata
// bcolor() sets background and border together
title("My title", box bcolor(gs14))
```

If you want a filled box **and** a visibly distinct border, prefer
`fcolor()` plus `lcolor()` rather than `bcolor()` alone.

### Trap 8 — `span` matters when notes/captions coexist with legends

Without `span`, `note()` and `caption()` often align to the plot region only.
This can look off when the graph includes a large right-side legend or wide margins.

```stata
note("Source: Authors' calculations", span)
caption("Outcome measured in logs", span)
```

---

## 8. Publication Templates

### Event-study title block + source note

```stata
twoway ///
    (rcap ci_hi ci_lo rel_year, lcolor(gs8) msize(small)) ///
    (scatter beta rel_year, mcolor(navy) msymbol(circle) msize(small)), ///
    yline(0, lcolor(gs10) lpattern(dash)) ///
    xline(-1, lcolor(gs8) lpattern(shortdash)) ///
    xlabel(-6(1)6) ///
    ylabel(, format(%9.2f) angle(0)) ///
    title("Dynamic treatment effects") ///
    subtitle("Reference period: year -1") ///
    note("Source: Authors' calculations. Bars denote 95% confidence intervals.", span) ///
    legend(off)
```

### In-plot annotation box

```stata
twoway ///
    (line outcome year, lcolor(navy) lwidth(medthick)) ///
    , ///
    title("Outcome over time") ///
    text(48.5 1923 ///
         "The 1918 influenza pandemic was the worst epidemic" ///
         "known in the US." ///
         "More citizens died than in all combat deaths of the" ///
         "20th century.", ///
         placement(se) ///
         box fcolor(gs15) lcolor(gs10) lwidth(thin) ///
         justification(left) ///
         margin(l+4 r+2 t+1 b+1) ///
         width(85))
```

### Scatter with selective marker labels

```stata
generate pos = 3
replace pos = 9 if stateab == "DC"
replace pos = 12 if stateab == "NY"

twoway ///
    (scatter ownhome propval100, ///
        mlabel(stateab) ///
        mlabvposition(pos) ///
        mlabsize(small) ///
        mlabcolor(black) ///
        msymbol(Oh) mcolor(navy)), ///
    xtitle("Percent homes over $100K") ///
    ytitle("Percent who own home")
```

### Text instead of mlabel for a single outlier

```stata
twoway ///
    (scatter propval100 ownhome, msymbol(Oh) mcolor(navy)) ///
    , ///
    text(62 45 "DC", placement(e) size(medsmall))
```

### Full-width boxed title

```stata
twoway scatter y x, ///
    title("Treatment effects by subgroup" ///
          "With county and year fixed effects", ///
          box bexpand span justification(left) ///
          fcolor(gs15) lcolor(gs10) margin(medsmall))
```
