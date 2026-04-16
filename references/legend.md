# legend.md — Legends in Stata Graphics

Source: Stata 19 Graphics Reference Manual [G-3] `legend_options` (pp.574-592);
Mitchell (2022) §8.9.

---

## 1. Overview

Stata legends are not just on/off switches. They control four separate things:

| Task | Main suboptions |
|------|-----------------|
| Decide which keys appear | `order()` `label()` `lastlabel()` `all` |
| Lay out the keys | `rows()` `cols()` `colfirst` `textfirst` `stack` |
| Style the legend box | `size()` `symxsize()` `symysize()` `textwidth()` `region()` `bmargin()` |
| Place the legend | `position()` `ring()` `bplacement()` `span` `at()` |

**Decision rule:**
- Keep the legend if it carries real information not already obvious from labels or panel titles.
- Turn it off with `legend(off)` when the graph is self-labeling or the legend wastes space.
- If the legend is necessary but too wide on the right, move it to the bottom or convert it into a narrow vertical key.

**Division of labor:**
- Legend text and placement belong here.
- Legend titles, subtitles, notes, and captions reuse `title_options`; see [labels-text.md](./labels-text.md).
- `by()` layout is only discussed here insofar as it changes legend placement. Full `by()` coverage belongs in `by-over.md`.

---

## 2. Syntax: `legend(contents, location)`

### 2.1 Standard syntax

```stata
legend([contents] [location])
```

For contour-line plots, Stata also allows:

```stata
plegend([contents] [location])
```

The official manual explicitly splits legend control into two groups:

| Group | What it controls |
|-------|------------------|
| `contents` | Which keys appear, their text, layout, sizing, titles, and box styling |
| `location` | Whether the legend is shown and where it is placed |

This distinction matters in ordinary `twoway` graphs, and it matters even more with `by()`.

### 2.2 Content suboptions

```stata
order(orderinfo)
label(# "text" ["text" ...])
lastlabel(# "text" ["text" ...])
holes(numlist)
all

style(legendstyle)
rows(#) cols(#)
colfirst
textfirst
stack
rowgap(size) colgap(size)
symplacement(compassdirstyle)
keygap(size)
symysize(size) symxsize(size)
textwidth(size)
forcesize
bmargin(marginstyle)

textbox_options
title_options
region(...)
```

### 2.3 Location suboptions

```stata
off | on
position(clockposstyle)
ring(ringposstyle)
bplacement(compassdirstyle)
span
at(#)
```

---

## 3. When Legends Appear and When to Turn Them Off

Stata shows the standard legend whenever the graph uses more than one distinct symbol, broadly defined to include:
- markers
- line styles
- color swatches

With a single symbol, Stata still constructs a legend internally, but it does not display it unless forced.

```stata
// One symbol: legend exists but is hidden
line le year, legend(on)

// More than one symbol: legend appears by default
line le_m le_f year

// Suppress it
line le_m le_f year, legend(off)
```

**Practical rule:**
- Use `legend(on)` only when a one-series graph still needs an explicit label.
- Use `legend(off)` when labels, panel titles, or direct annotations already identify the series.

---

## 4. Contents Control

### 4.1 `order()`: show, suppress, reorder, and insert text

`order()` is the main tool for deciding which keys appear and in what sequence.

```stata
// Default order for three keys
legend(order(1 2 3))

// If there were four keys, drop the fourth
legend(order(1 2 3))

// Reorder
legend(order(2 1 3))
```

`order()` can also insert text into the legend by using a dash:

```stata
// Insert a group heading
legend(order(1 - "Predicted" 2 3))

// Insert a blank line, then a heading
legend(order(1 - " " "Predicted" 2 3))
```

You can also override the displayed label directly inside `order()`:

```stata
legend(order(1 "Observed 1992" 2 3))
```

The manual notes that this works, but it is usually cleaner to use `label()` for relabeling and reserve `order()` for structure.

### 4.2 `label()`: relabel a specific key

```stata
line le_m le_f year, ///
    legend(label(1 "Male") label(2 "Female"))
```

Multiline labels are allowed:

```stata
legend(label(1 "Observed" "1992-1993"))
```

One `label()` suboption changes one key. Use multiple `label()` suboptions for multiple keys.

### 4.3 `lastlabel()`: relabel after `order()` has already changed things

`lastlabel()` is easy to skip, but it matters when Stata or the user has already modified labels through `order()`.

```stata
legend(order(5 "Key 5" 4 "Key 4") ///
       lastlabel(1 "Baseline"))
```

Key point:
- `label()` refers to the original key index.
- `lastlabel()` refers to the final key index **as displayed on the graph**.

This is especially useful after `marginsplot`, where Stata often uses `order()` internally.

### 4.4 `holes()`: blank slots in multirow/multicolumn legends

```stata
legend(rows(2) holes(2))
```

`holes()` works only when the keys are arranged in more than one row and more than one column.
The official manual explicitly says there is always an `order()` solution that can reproduce the same result, and `order()` is more general.

**Recommendation:**
- Use `holes()` for simple grid cleanup.
- Use `order()` when you need full control or inserted text.

### 4.5 `all`: keep separate keys even when styles repeat

By default, Stata may collapse keys that share the same overall plot style.

```stata
scatter ylow yhigh x, pstyle(p1 p1) legend(all)
```

Use `all` when separate series need separate keys even if they share a symbol or line style.

---

## 5. Layout and Ordering

### 5.1 `rows()` and `cols()`

These determine the legend table shape.

```stata
// Vertical legend
legend(cols(1))

// Horizontal legend
legend(rows(1))

// Bottom legend with two columns
legend(position(6) cols(2))
```

Typical use:
- `cols(1)` for a narrow vertical legend
- `rows(1)` for a horizontal legend below the plot
- `cols(2)` as a compact default when the legend moves to the bottom

### 5.2 `colfirst`

When keys span multiple columns, `colfirst` changes the reading order from across-then-down to down-then-across.

Use it when the semantic grouping is vertical rather than horizontal.

### 5.3 `textfirst`

`textfirst` places descriptive text before the symbol:

```stata
legend(textfirst)
```

This is especially useful for narrow legends or stacked layouts where the symbol should align at the right or bottom edge.

### 5.4 `stack`

`stack` places the symbol and text vertically instead of side-by-side.

```stata
legend(stack)
```

Mitchell and the official manual both treat this as most useful for narrow single-column legends:

```stata
legend(cols(1) stack symplacement(left) symxsize(13) forcesize rowgap(4))
```

That pattern is one of the most useful publication layouts when the right-hand legend is too wide.

### 5.5 `rowgap()` and `colgap()`

These control spacing between rows and columns:

```stata
legend(rowgap(2) colgap(3))
```

Use them when:
- multiline labels look cramped
- columns are too far apart
- a bottom legend needs tighter packing

---

## 6. Sizing and Styling the Legend Box

### 6.1 Symbol and text geometry

These options control the dimensions of the key itself:

| Option | Role |
|--------|------|
| `symxsize()` | Width allocated to symbol area |
| `symysize()` | Height allocated to symbol area |
| `textwidth()` | Width allocated to descriptive text |
| `keygap()` | Gap between symbol area and text |
| `symplacement()` | Alignment of the symbol within its allotted area |
| `forcesize` | Respect `symxsize()` / `symysize()` exactly |

Example:

```stata
legend(symxsize(10) symysize(5) textwidth(20) keygap(2))
```

Important detail from the manual:
- Without `forcesize`, Stata may compress or expand the symbol area after looking at the actual symbols.
- With `forcesize`, the requested symbol sizes are respected.

### 6.2 `bmargin()` vs `region(margin())`

These two margins do different jobs:

| Option | Meaning |
|--------|---------|
| `bmargin()` | Outer margin around the whole legend |
| `region(margin())` | Inner margin between border and legend contents |

This distinction is easy to miss.

```stata
legend(bmargin(small) region(margin(medsmall)))
```

Use `bmargin()` when nearby graph elements crowd the legend.
Use `region(margin())` when text feels too close to the legend border.

### 6.3 Text appearance via textbox options

Legend key text accepts `textbox_options`, especially:

```stata
legend(size(small) color(black))
```

Most common use:
- `size()` to reduce legend text in dense figures
- occasionally `color()` or justification-related formatting

For the full textbox system, see [labels-text.md](./labels-text.md).

### 6.4 Legend titles and notes

Legends accept `title()`, `subtitle()`, `note()`, and `caption()`.

```stata
legend(cols(1) subtitle("Legend"))
```

Mitchell’s advice matches the official examples: `subtitle()` is often better than `title()` because `title()` usually looks too large inside a legend.

### 6.5 `region()`: border and background

`region()` controls the legend box itself:

```stata
legend(region(lcolor(black)))
legend(region(lstyle(none)))
legend(region(fcolor(gs15)))
```

Useful suboptions:

| Suboption | Use |
|-----------|-----|
| `lcolor()` | Border color |
| `lwidth()` | Border thickness |
| `lpattern()` | Border pattern |
| `fcolor()` | Background fill |
| `color()` | Border + fill together |
| `margin()` | Inner padding |
| `style()` | Start from a preset areastyle |

Common publication choices:
- `region(lstyle(none))` for no border
- `region(fcolor(none) lcolor(none))` for a fully invisible box
- `region(fcolor(white) lcolor(gs8))` for a clear boxed legend inside the plot

---

## 7. Placement

### 7.1 Defaults

Under the default `stcolor` scheme, the standard legend appears at:

```stata
position(3) ring(4)
```

That means: to the right of the plot region, fairly far out.

### 7.2 `position()` and `ring()`

`position()` uses the clock system:

| Value | Meaning |
|-------|---------|
| `12` | above |
| `3` | right |
| `6` | below |
| `9` | left |

`ring()` controls distance from the plot region:
- `ring(0)` puts the legend inside the plot region
- larger positive values move it farther out

```stata
// Bottom legend
legend(position(6))

// Inside lower-right corner
legend(position(5) ring(0))
```

### 7.3 `bplacement()`

`bplacement()` matters only when `ring(0)` is used. It refines where the legend sits inside the plot region.

```stata
legend(ring(0) bplacement(seast))
```

This is a precise way to anchor an in-plot legend without relying on trial and error.

### 7.4 `span`

`span` changes whether the legend is centered relative to the plot region or the full graph area.

This is most useful when:
- the graph has wide margins
- the graph has a right-side axis or large note
- a bottom legend should center under the full figure rather than only under the plot region

### 7.5 Practical placement rules

- Right side: good for short labels and a small number of series.
- Bottom: good for long labels, especially with `rows(1)` or `cols(2)`.
- Inside plot: good when the plot has empty space and outside margins are tight.

---

## 8. `legend()` with `by()`

This is the most important structural rule in the file.

When `by()` is used:
- **content** suboptions stay outside `by()`
- **location** suboptions go inside `by()`

Official example:

```stata
scatter mpg weight || lfit mpg weight ||, ///
    legend(order(2)) ///
    by(foreign, total legend(position(4)))
```

Why:
- the plots must know which keys they are producing when they are drawn
- the final assembled by-graph decides where the common legend is placed

If you write this instead:

```stata
scatter mpg weight || lfit mpg weight ||, ///
    legend(order(2) position(4)) ///
    by(foreign, total)
```

the `position(4)` part is ignored.

### 8.1 `at(#)`

With `by()`, `at(#)` places the legend in a specific cell of the `R x C` array:

```stata
scatter mpg weight || lfit mpg weight ||, ///
    legend(order(2)) ///
    by(foreign, total legend(at(4) position(0)))
```

This is useful when one cell is intentionally left open for the legend.

### 8.2 Turning the legend off in `by()`

To suppress the legend in a by-graph, put it inside `by()`:

```stata
by(groupvar, legend(off))
```

Not:

```stata
legend(off) by(groupvar)
```

when the issue is the assembled by-graph legend.

### 8.3 `by()` bottom-legend patterns

Common approaches:

```stata
by(groupvar, row(1) legend(position(6)))
```

or use a scheme / bystyle that already moves the legend downward, such as the official examples using `scheme(stcolor_alt)` or `style(altleg)`.

---

## 9. Common Traps

### 9.1 `order()` indexes start from 1

Legend keys are numbered `1, 2, 3, ...`, not zero-based.

### 9.2 `holes()` is limited

`holes()` works only for legends with more than one row and more than one column. If it does nothing, use `order()` instead.

### 9.3 `label()` and `lastlabel()` are not interchangeable

- `label()` targets the original key index
- `lastlabel()` targets the final displayed key index

After `order()`, this distinction matters.

### 9.4 Long labels can distort graph layout

The official manual highlights three recurrent problems:
- text can spill outside the legend border
- a bottom legend can widen the plot region and collide with y-axis labels/title
- a right-side legend with long text can make the graph too narrow

Standard fixes:
- shorten labels
- split labels across lines
- move the legend to the bottom
- add `region(margin())`
- convert to a narrow vertical legend with `cols(1) stack ...`

### 9.5 `ring(0)` without `bplacement()` is often underspecified

If the legend is inside the plot, add `bplacement()` when you need a specific corner or side.

### 9.6 `by()` ignores misplaced location suboptions

With `by()`, `position()` / `at()` / `off` belong inside `by(... legend(...))`, not in the outer `legend()`.

---

## 10. Publication Templates

### 10.1 Clean bottom legend for two series

```stata
line y1 y2 x, ///
    legend(position(6) cols(2) region(lcolor(none) fcolor(none)))
```

Use when labels are moderately long and right-side placement wastes width.

### 10.2 In-plot boxed legend

```stata
line y1 y2 x, ///
    legend(position(5) ring(0) ///
           label(1 "Treatment") label(2 "Control") ///
           region(fcolor(white) lcolor(gs8)))
```

Use when the plot has unused white space.

### 10.3 Narrow vertical legend

```stata
twoway ///
    (scatter y x) ///
    (lfit y x) ///
    (qfit y x), ///
    legend(cols(1) stack ///
           symplacement(left) symxsize(13) forcesize rowgap(4))
```

Use when the legend needs to stay at the side but must not become too wide.

### 10.4 `by()` graph with single fitted-line legend

```stata
twoway ///
    (scatter mpg weight) ///
    (lfit mpg weight), ///
    legend(order(2) label(2 "Fitted values")) ///
    by(foreign, total row(1) legend(position(6)))
```

Use when only one layer needs to be identified across panels.

---

## 11. Final Guidance

For publication-quality graphs, the most reliable sequence is:

1. Decide whether the legend is needed at all.
2. Trim the contents with `order()` and `label()`.
3. Choose layout with `rows()` / `cols()` / `stack`.
4. Fix spacing and box styling with `region()`, `bmargin()`, and size controls.
5. Only then fine-tune placement with `position()` / `ring()` / `bplacement()`.

That order matches how legend problems usually arise in real work: content first, geometry second, placement last.
