# Markers in Stata Graphics

Sources: [G-3] marker_options, [G-4] symbolstyle, markerstyle, markersizestyle, size — Stata 19 Graphics Reference Manual

---

## marker_options — Full Syntax

```stata
msymbol(symbolstyle)        // shape of marker
mcolor(colorstyle)          // color of marker (fill + outline together)
msize(markersizestyle)      // size of marker
msangle(anglestyle)         // rotation angle (rarely used)
mfcolor(colorstyle)         // fill color only
mlcolor(colorstyle)         // outline (line) color only
mlwidth(linewidthstyle)     // outline line width
mlalign(linealignmentstyle) // outline alignment: inside | outside | center
mlstyle(linestyle)          // outline line style
mstyle(markerstyle)         // preset marker style (overrides other options)
pstyle(pstyle)              // preset plot style
recast(newplottype)         // change plot type after rendering
```

**Option precedence:** `mstyle()` sets shape + size + color + fill + outline as a bundle;
individual options (`msymbol`, `mcolor`, etc.) override specific attributes of that bundle.
`mcolor()` is a shorthand that sets both `mfcolor()` and `mlcolor()` together.

---

## Symbol Shapes (symbolstyle)

Specified via `msymbol()`. Synonyms are single-letter or two-letter abbreviations.

### Large symbols

| symbolstyle       | Synonym | Description         |
|-------------------|---------|---------------------|
| `circle`          | `O`     | solid circle        |
| `diamond`         | `D`     | solid diamond       |
| `triangle`        | `T`     | solid triangle      |
| `square`          | `S`     | solid square        |
| `plus`            | `+`     | plus sign           |
| `X`               | `X`     | X mark              |
| `arrowf`          | `A`     | filled arrowhead    |
| `arrow`           | `a`     | hollow arrowhead    |
| `pipe`            | `\|`    | vertical bar        |
| `V`               | `V`     | inverted triangle   |

### Small symbols (sm prefix)

| symbolstyle       | Synonym | Description         |
|-------------------|---------|---------------------|
| `smcircle`        | `o`     | small solid circle  |
| `smdiamond`       | `d`     | small solid diamond |
| `smsquare`        | `s`     | small solid square  |
| `smtriangle`      | `t`     | small solid triangle|
| `smplus`          | —       | small plus          |
| `smx`             | `x`     | small X             |
| `smv`             | `v`     | small inverted triangle |

### Hollow symbols

| symbolstyle         | Synonym | Description              |
|---------------------|---------|--------------------------|
| `circle_hollow`     | `Oh`    | hollow circle (large)    |
| `diamond_hollow`    | `Dh`    | hollow diamond (large)   |
| `triangle_hollow`   | `Th`    | hollow triangle (large)  |
| `square_hollow`     | `Sh`    | hollow square (large)    |
| `smcircle_hollow`   | `oh`    | hollow circle (small)    |
| `smdiamond_hollow`  | `dh`    | hollow diamond (small)   |
| `smtriangle_hollow` | `th`    | hollow triangle (small)  |
| `smsquare_hollow`   | `sh`    | hollow square (small)    |

### Special

| symbolstyle | Synonym | Description         |
|-------------|---------|---------------------|
| `point`     | `p`     | tiny dot            |
| `none`      | `i`     | invisible marker    |

**View symbol palette:**
```stata
palette symbolpalette [, scheme(schemename)]
```

**Query available symbolstyles:**
```stata
graph query symbolstyle
```

---

## Hollow vs Filled Markers

A hollow marker is a filled marker with `mfcolor(none)`. These are equivalent:

```stata
msymbol(Oh)                      // shorthand: hollow circle
msymbol(O) mfcolor(none)         // explicit: filled circle, no fill

msymbol(Dh)
msymbol(D) mfcolor(none)
```

**Custom fill with outline:**
```stata
// Dark outline, lighter fill — creates two-tone effect
msymbol(O) mlcolor(gs5) mfcolor(gs12)
msymbol(O) mlcolor(navy) mfcolor(navy%30)
```

Small vs large symbols differ only in `msize()`:
```stata
msymbol(O) msize(medium)   // equivalent to msymbol(O)
msymbol(o)                 // equivalent to msymbol(O) msize(small)
```

---

## Marker Size (markersizestyle)

### Named sizes

```
vtiny  tiny  vsmall  small  medsmall  medium  medlarge  large  vlarge  huge  vhuge  ehuge
```

Default size for most schemes is `small`.

### Numeric and unit sizes

Sizes follow the general `size` syntax:

| Format  | Meaning                                             | Example           |
|---------|-----------------------------------------------------|-------------------|
| `#`     | scheme-dependent units (relative since Stata 16)    | `msize(2)`        |
| `#pt`   | printer points                                      | `msize(4pt)`      |
| `#in`   | inches                                              | `msize(.1in)`     |
| `#cm`   | centimeters                                         | `msize(.2cm)`     |
| `#rs`   | relative size: % of min(graph width, graph height)  | `msize(1.5rs)`    |
| `*#`    | multiply current size by #                          | `msize(*1.5)`     |

```stata
msize(small)       // named
msize(2)           // numeric (relative)
msize(*1.5)        // 1.5× the default size
msize(*.5)         // half the default size
msize(4pt)         // absolute: 4 printer points
```

---

## Marker Color Options

```stata
mcolor(colorstyle)     // sets both fill and outline color at once
mfcolor(colorstyle)    // fill color only
mlcolor(colorstyle)    // outline color only
mlwidth(linewidthstyle) // outline line thickness
```

**Common patterns:**
```stata
// Solid colored marker
msymbol(O) mcolor(navy)

// Hollow with colored outline
msymbol(Oh) mlcolor(red)

// Semi-transparent fill, colored outline
msymbol(O) mfcolor(blue%30) mlcolor(blue)

// No outline
msymbol(O) mfcolor(navy) mlwidth(none)
```

Opacity via `%`: `mcolor(navy%50)` = navy at 50% opacity.

---

## Marker Style Presets (markerstyle)

`mstyle()` selects a predefined combination of symbol + size + color + fill + outline.

### Plot marker families

| markerstyle      | Used by                          |
|------------------|----------------------------------|
| `p1`–`p15`       | first–fifteenth plot in a graph  |
| `p1box`–`p15box` | box chart markers                |
| `p1dot`–`p15dot` | dot chart markers                |

Specific attributes of the preset can be overridden after specifying `mstyle()`:

```stata
scatter y x, mstyle(p1) mcolor(red)   // use p1 preset, override color
```

---

## Multiple Plots: symbolstylelist

When overlaying multiple plots, `msymbol()` accepts a space-separated list:

```stata
scatter mpg1 mpg2 weight, msymbol(O d)    // circle for mpg1, diamond for mpg2
scatter mpg1 mpg2 weight, msymbol(Oh .)   // hollow circle, then default
```

**Stylelist shorthands** (see [G-4] stylelists):
- `.` — use the default for this element
- `=` — repeat the previous element
- `..` — repeat previous to end of list

```stata
// Three plots: O, then default, then default
msymbol(O . .)

// Three plots: O d O (repeat O at end via default)
msymbol(O d p)
```

---

## Common Patterns for Economics Figures

### Event study: filled circle at zero, hollow elsewhere
```stata
scatter b t, msymbol(Oh) msize(small) mcolor(navy)
// Mark period 0 separately
scatter b0 t if t==0, msymbol(O) msize(small) mcolor(navy) nodraw
```

### Coefficient plot: differentiate two models
```stata
twoway ///
  (scatter b1 y, msymbol(O) msize(small) mcolor(navy)) ///
  (scatter b2 y, msymbol(D) msize(small) mcolor(maroon))
```

### Remove markers (line only)
```stata
twoway line y x, msymbol(none)    // or simply: twoway line y x
```

### Semi-transparent markers for overplotting
```stata
scatter y x, msymbol(O) mcolor(navy%40) msize(medsmall)
```
