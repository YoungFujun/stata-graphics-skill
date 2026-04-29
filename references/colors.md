# Colors in Stata Graphics

Sources: [G-4] colorstyle — Stata 19 Graphics Reference Manual; graph-schemes.md (palettes package)

---

## Color Specification Syntax

```stata
// Set color of a specific object
mcolor(colorstyle)
lcolor(colorstyle)
fcolor(colorstyle)

// Set opacity only (keeps default color, 0=transparent, 100=opaque)
mcolor(%30)

// Set color and opacity together
mcolor(navy%50)
lcolor("26 71 111"%70)

// Adjust intensity (brightness) of default color
bcolor(*0.8)         // lighter (< 1)
mcolor(*1.5)         // darker (> 1)

// Set color and intensity
mcolor(green*0.8)
```

---

## Named Colors

### Grayscale (gs0–gs16)

`gs0` = black, `gs16` = white, `gs8` ≈ medium gray. Other notes: `gray` = `gs8`, `dimgray` ≈ between gs14 and gs15.

```stata
mcolor(gs0)    // black
mcolor(gs4)    // dark gray
mcolor(gs8)    // medium gray  (same as gray)
mcolor(gs12)   // light gray
mcolor(gs16)   // white
```

Use **different gs values + different line patterns** for multi-series grayscale/print graphs.

### Standard named colors

```
black       white
blue        bluishgray   brown        cranberry    cyan
dimgray     dkgreen      dknavy       dkorange     eggshell
emerald     forest_green gold         gray         green
khaki       lavender     lime         ltblue       ltbluishgray
ltkhaki     magenta      maroon       midblue      midgreen
mint        navy         olive        olive_teal   orange
orange_red  pink         purple       red          sand
sandb       sienna       stone        teal         yellow
```

### Special values

| colorstyle         | Meaning                                    |
|--------------------|--------------------------------------------|
| `none`             | no color; invisible; draws nothing         |
| `background` / `bg`| same color as graph background             |
| `foreground` / `fg`| same color as graph foreground             |
| `stc1`–`stc15`     | colors 1–15 of the `stcolor` scheme        |
| `stblue` / `stgreen` / `stred` / `styellow` | specific `stcolor` scheme colors |

### Economist magazine colors

```
ebg       (background)   ebblue   (bright blue)
edkblue   (dark blue)    eltblue  (light blue)
eltgreen  (light green)  emidblue (midblue)
erose     (rose)
```

**Preview all named colors:**
```stata
graph query colorstyle       // text list
palette color navy           // visual swatch for one color
```

---

## Color Specification Formats

All of these can be used anywhere a `colorstyle` is accepted.
**Always enclose RGB, CMYK, HSV, and hex values in quotes.**

### RGB — `"R G B"`

Three integers, each 0–255.

```stata
mcolor("0 0 255")          // blue
mcolor("26 71 111")        // navy equivalent
mcolor("255 255 255")      // white
mcolor("128 128 128")      // gray
```

### CMYK — `"C M Y K"`

Four values: either integers 0–255 or proportions 0.0–1.0.
If all four values are ≤ 1, they are treated as proportions.

```stata
mcolor("0 255 255 0")      // red  (integer form)
mcolor("0 1 1 0")          // red  (proportion form)
mcolor(".334 .157 0 .565") // navy (proportion form)
```

Note: Stata normalizes CMYK by reducing CMY until one value = 0, adding the remainder to K.
Colors returned by `graph export` may differ from what you entered.

### HSV — `"hsv H S V"`

H: hue 0–360 (degrees), S: saturation 0–1, V: value/brightness 0–1.
Must be prefixed with `hsv`.

```stata
mcolor("hsv 0 1 1")        // red
mcolor("hsv 240 1 1")      // blue
mcolor("hsv 209 .766 .435")// navy equivalent
mcolor("hsv 0 0 1")        // white
```

### Hexadecimal — `"#RRGGBB"`

Six hex digits, uppercase or lowercase.

```stata
mcolor("#FF0000")          // red
mcolor("#00FF00")          // green
mcolor("#0000FF")          // blue
mcolor("#FFFFFF")          // white
mcolor("#000000")          // black
mcolor("#1A476F")          // navy (Stata's named navy ≈ RGB 26 71 111)
```

---

## Opacity

Opacity is a percentage of full coverage (100 = fully opaque, 0 = fully transparent).

```stata
// Append % to any colorstyle
mcolor(navy%50)              // navy at 50% opacity
mcolor("0 255 0"%50)         // RGB green at 50% opacity
lcolor("#FF0000"%80)         // hex red at 80% opacity

// %# alone: use default color at # opacity
fcolor(%30)                  // default fill color at 30% opacity

// color(%0) = fully transparent = equivalent to color(none)
```

**Common uses in economics figures:**
```stata
// Semi-transparent CI band
rarea ub lb t, fcolor(navy%15) lwidth(none)

// Overlapping scatter
scatter y x, mcolor(navy%40) msymbol(O)

// Two-color fill for CI comparison
rarea ub1 lb1 t, fcolor(navy%20) lwidth(none)
rarea ub2 lb2 t, fcolor(maroon%20) lwidth(none)
```

---

## Intensity (Brightness)

Intensity multiplies the color's brightness: >1 makes it darker, <1 makes it lighter.
`color*0` = white; `color*255` = maximum darkness.

```stata
mcolor(green*.8)      // slightly lighter green
mcolor(navy*1.2)      // slightly darker navy
mcolor(*0.7)          // default color, 30% lighter

// Intensity with RGB
mcolor("0 255 255*1.2")  // valid
```

**Intensity vs opacity:** These are fundamentally different.
- Opacity: controls transparency (how much background shows through)
- Intensity: changes the actual color value (lighter/darker, fully opaque)

---

## Multi-Plot: colorstylelist

When multiple series share an option, provide a space-separated list (see [G-4] stylelists):

```stata
twoway line y1 y2 y3 x, lcolor(navy maroon forest_green)
scatter y1 y2 x,        mcolor(navy%60 maroon%60)
```

Shorthands:
- `.` — default for this element
- `=` — repeat previous
- `..` — repeat previous to end

```stata
lcolor(navy . .)       // navy for first, default for rest
mcolor(navy = =)       // navy for all three
```

---

## Export and CMYK

`graph export` stores colors as RGB + opacity values internally.

For journals requiring CMYK (PostScript / EPS / PDF):

```stata
// Per-export
graph export figure.eps, cmyk(on) replace

// Permanent setting for PostScript
translator set Graph2ps cmyk on
// Permanent setting for EPS
translator set Graph2eps cmyk on
```

**Note:** Stata normalizes CMYK on input, so exported CMYK values may differ from what you specified. Use RGB for screen output; switch to CMYK only when the publisher requires it.

---

## Community Package: palettes (Ben Jann)

The `palettes` package provides access to standard palette collections via the `colorpalette` command. Required for `grstyle set color` and for programmatic color access.

**Note:** `colorpalette` requires both `palettes` and `colrspace` — install both, neither works alone.

```stata
ssc install palettes, replace
ssc install colrspace, replace   // required dependency
```

### Preview palettes visually

```stata
colorpalette tableau            // visual display
colorpalette viridis
colorpalette Set1               // ColorBrewer
colorpalette okabe              // Okabe & Ito (colorblind-safe)
colorpalette tol bright         // Paul Tol (colorblind-safe)
```

### Get colors programmatically

```stata
colorpalette tableau, nograph   // stores colors in r(p)
local colors `r(p)'

// Access individual colors
local c1 : word 1 of `colors'
local c2 : word 2 of `colors'
twoway (scatter y1 x, mcolor("`c1'")) (scatter y2 x, mcolor("`c2'"))
```

### Generate n colors from a palette

```stata
colorpalette viridis, n(5) nograph
local colors `r(p)'
```

### Palette types

| Type        | Palettes                        | Best for                          |
|-------------|---------------------------------|-----------------------------------|
| Qualitative | `tableau`, `Set1`, `okabe`, `tol bright` | Categorical groups           |
| Sequential  | `viridis`, `Blues`, `Reds`      | Ordered/continuous data           |
| Diverging   | `RdBu`, `PiYG`, `BrBG`         | Data with meaningful midpoint     |

### Use with grstyle

```stata
grstyle init
grstyle set color tableau               // named palette
grstyle set color okabe                 // colorblind-safe
grstyle set color "0 114 178" "213 94 0" "0 158 115"  // manual RGB
```

---

## Publication Color Recommendations (Economics)

### Single-color figures (most common)

```stata
// Coefficient plots, event studies — one series
lcolor(navy) mcolor(navy)
```

### Two-series comparison

```stata
// Option A: navy + maroon (high contrast, B&W friendly when different symbols used)
mcolor(navy maroon)

// Option B: colorblind-safe (Okabe & Ito first two colors)
mcolor("0 114 178" "213 94 0")    // blue + vermillion
```

### Multi-series (3–6 groups)

```stata
// Tableau (clean, professional, distinct)
colorpalette tableau, n(#) nograph
local colors `r(p)'

// Okabe & Ito (8 colorblind-safe colors)
colorpalette okabe, nograph
```

### Grayscale / monochrome print

```stata
// Use gs values spaced to be distinguishable in print
// Plus vary line patterns and marker symbols
twoway ///
  (line y1 x, lcolor(gs2)  lpattern(solid) lwidth(medthick)) ///
  (line y2 x, lcolor(gs8)  lpattern(dash)  lwidth(medthick)) ///
  (line y3 x, lcolor(gs13) lpattern(dot)   lwidth(medthick))
```

### Confidence intervals / shaded bands

```stata
// Always use low opacity fill (15–25%) + no outline
fcolor(navy%15) lwidth(none)

// For two overlapping CI bands
fcolor(navy%20) lwidth(none)      // series 1
fcolor(maroon%20) lwidth(none)    // series 2
```
