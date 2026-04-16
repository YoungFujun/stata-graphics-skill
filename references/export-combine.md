# export-combine.md — Saving, Combining, and Exporting Stata Graphs

Source: Stata 19 Graphics Reference Manual [G-3] `saving_option` (pp.637-638),
`eps_options` (pp.564-566), `pdf_options` (pp.611-613), `png_options` (p.615),
`svg_options` (pp.644-647), `tif_options` (p.657); [G-2] `graph combine`
(pp.106-114), `graph display` (pp.121-128), `graph export` (pp.148-151), and
`graph manipulation` excerpts on `graph save` / `graph use`.

---

## 1. Overview: Save -> Redisplay -> Combine -> Export

In Stata, graph output is usually a four-step workflow:

1. Draw the graph.
2. Save it as a `.gph` object for reuse.
3. Redisplay or combine it if needed.
4. Export the final graph to PDF, PNG, EPS, SVG, TIFF, etc.

This file focuses on that workflow, not on the styling of the graph itself.

**Decision rule:**
- Use `.gph` files for reusable intermediate graphs.
- Use `graph combine` when panels must be drawn separately and assembled later.
- Use vector export (`pdf`, `eps`, `svg`) for line art and publication workflows.
- Use raster export (`png`, `tif`) when the target is slides, Word, or a publisher requiring bitmap files.

**Division of labor:**
- Graph styling belongs in the other reference files.
- Legend control belongs in [legend.md](./legend.md).
- `by()` is usually preferable to `graph combine` when all panels are the same plot type and share one design. The full comparison belongs in `by-over.md`.

---

## 2. Saving Graphs: `saving()` vs `graph save`

### 2.1 `saving()`

Use `saving()` when you want to save a graph at the moment it is drawn.

```stata
twoway scatter y x, saving(mygraph)
twoway scatter y x, saving(mygraph, replace)
```

Syntax:

```stata
saving(filename [, asis replace])
```

If no extension is given, Stata assumes `.gph`.

### 2.2 `graph save`

Use `graph save` after the graph already exists in memory.

```stata
twoway scatter y x
graph save mygraph
```

Officially, `saving()` and `graph save` create the same `.gph` result. The difference is workflow:

| Method | Best use |
|--------|----------|
| `saving()` | You do not want to forget to save |
| `graph save` | You want to edit or inspect the graph first |

### 2.3 Live vs as-is graphs

This is the most important conceptual distinction in graph saving.

| Format | Meaning |
|--------|---------|
| live | Default; graph can still be edited and can change appearance with scheme changes |
| `asis` | Frozen appearance; cannot be edited or re-schemed like a live graph |

```stata
twoway scatter y x, saving(mygraph, asis)
graph save mygraph, asis
```

The manual’s rule is explicit:
- live format is flexible and reusable
- `asis` is more like a frozen picture

**Recommendation:**
- Save working graphs in live format unless you have a specific reason to freeze them.
- Use `asis` only when you need exact portability of appearance or a stable archival snapshot.

### 2.4 Paths and replacement

```stata
saving("results/figures/event_study_main", replace)
```

Use `replace` only when overwriting is intended.

---

## 3. Loading and Redisplaying Graphs

### 3.1 `graph use`

`graph use` loads a graph from disk into memory and displays it.

```stata
graph use mygraph.gph
graph use mygraph.gph, name(panel1)
```

This is the standard bridge from saved `.gph` files to later combination or export.

### 3.2 `graph display`

`graph display` redisplays a graph already stored in memory.

```stata
graph display panel1
graph display panel1, ysize(2) xsize(3)
graph display panel1, scale(1.2)
graph display panel1, scheme(sj)
```

Useful options:

| Option | Role |
|--------|------|
| `ysize()` / `xsize()` | Change overall graph size in inches |
| `margins()` | Change outer margins |
| `scale()` | Resize text, markers, and line widths together |
| `scheme()` | Reapply a scheme to the stored live graph |

### 3.3 Why redisplay matters

`graph display` is useful when:
- you need to test aspect ratios without rebuilding a complex graph
- you want to tweak margins before export
- you want to try a different scheme on a live graph

The manual is also clear about the limitation:
- changing schemes after the fact works best when the original graph did not hard-code unusual placements or named colors

### 3.4 Memory vs disk

| Storage | Persistence | Typical command |
|---------|-------------|-----------------|
| memory | temporary; lost when Stata exits or is cleared | `name()` `graph display` |
| disk (`.gph`) | persistent | `saving()` `graph save` `graph use` |

If the graph matters beyond the current session, save it to disk.

---

## 4. `graph combine`: Core Syntax

### 4.1 Basic syntax

```stata
graph combine name [name ...] [, options]
```

Each `name` can be:
- a graph in memory, such as `g1`
- a `.gph` file on disk, such as `g1.gph`
- a quoted disk name without suffix, such as `"g1"`

Examples:

```stata
graph combine g1 g2
graph combine g1.gph g2.gph
graph combine "g1" "g2"
```

### 4.2 Layout options

```stata
graph combine g1 g2 g3 g4, rows(2)
graph combine g1 g2 g3 g4, cols(2)
graph combine g1 g2 g3 g4, colfirst
graph combine g1 g2 g3, holes(2)
```

Key options:

| Option | Role |
|--------|------|
| `rows(#)` / `cols(#)` | Shape of the panel matrix |
| `colfirst` | Fill down columns instead of across rows |
| `holes(numlist)` | Leave specified cells blank |
| `title()` / `note()` / `caption()` | Add combined-graph titles |
| `name()` | Store combined graph in memory |
| `saving()` | Save combined graph to disk |

### 4.3 Memory graphs vs disk graphs

If you combine graphs in memory, use bare names:

```stata
twoway scatter y x, name(g1)
twoway line y x, name(g2)
graph combine g1 g2
```

If you combine graphs saved on disk, either include `.gph` or quote the names:

```stata
graph combine g1.gph g2.gph
graph combine "g1" "g2"
```

If you do neither, Stata assumes you meant memory graphs.

---

## 5. Preparing Subgraphs Before Combining

`graph combine` works best when the individual graphs have already been designed to align.

### 5.1 Use the same plotting logic

Before combining, keep these consistent across subgraphs whenever possible:
- axis ranges
- tick locations
- label sizes
- title logic
- legend treatment
- scheme

### 5.2 Pre-align the geometry

Common pre-combine cleanup:

```stata
ylabel(, angle(horizontal))
xscale(off)
graphregion(margin(...))
plotregion(margin(...))
```

The advanced `graph combine` example in the manual makes the point clearly: much of the alignment work happens **before** `graph combine`, not inside it.

### 5.3 `by()` may be better

Use `by()` instead of `graph combine` when:
- every panel is the same graph type
- scales should match automatically
- the layout is a standard small-multiples design

Use `graph combine` when:
- panels are heterogeneous
- some subgraphs must be drawn separately
- you need empty cells, unequal subgraph geometry, or different internal designs

---

## 6. Alignment and Size Control in `graph combine`

### 6.1 `xcommon` and `ycommon`

```stata
graph combine g1 g2 g3 g4, cols(2) ycommon xcommon
```

These force common scales across previously drawn `twoway` graphs.

Important limitations from the manual:
- they do not affect categorical axes in bar, box, and dot graphs
- in mixed-type combinations, they only affect graphs of the same type as the first graph combined

### 6.2 `iscale()`

`iscale()` rescales text and markers in the subgraphs:

```stata
graph combine g1 g2, iscale(1)
graph combine g1 g2, iscale(.8)
graph combine g1 g2, iscale(*1.2)
```

Rules:
- `iscale(1)` preserves original text/marker size
- `iscale(.5)` halves it
- `iscale(*#)` multiplies Stata’s default internal scaling rule

This is one of the highest-value combine options for publication output.

### 6.3 `altshrink`

`altshrink` uses a different scaling method:

```stata
graph combine g1 g2, altshrink
```

The manual describes it as drawing each graph at full size for its aspect ratio and then shrinking the whole result into the final panel area.

Use it when default scaling produces awkward line widths or text.

### 6.4 `imargin()`

```stata
graph combine g1 g2 g3 g4, imargin(0 0 0 0)
```

`imargin()` sets the margins around each subgraph inside the combined array.

Typical use:
- `imargin(0 0 0 0)` to push panels tightly together

Important consequence:
- if you specify `imargin()`, it overrides the original margin information that would otherwise influence subgraph geometry

### 6.5 `graphregion(margin())`

The official advanced example uses:

```stata
graph combine ..., imargin(0 0 0 0) graphregion(margin(l=22 r=22))
```

Interpretation:
- `imargin()` controls spacing between subgraphs
- `graphregion(margin())` helps control the overall aspect ratio of the combined graph

### 6.6 `fxsize()` and `fysize()`

This is the most important non-obvious rule in the file.

`fxsize()` and `fysize()` are specified **when drawing the subgraphs**, not in `graph combine`.

```stata
twoway histogram lexp, fraction horiz fxsize(25) saving(hy)
twoway histogram loggnp, fraction fysize(25) saving(hx)
graph combine hy.gph yx.gph hx.gph, holes(3) imargin(0 0 0 0)
```

The manual’s logic:
- `xsize()` / `ysize()` control physical size of an ordinary graph
- `graph combine` discards that physical-size information
- `fxsize()` / `fysize()` are the right tools when you need unequal subgraph areas or when `imargin()` is being used

### 6.7 What `graph combine` does not preserve

Do not assume that:
- the original physical `xsize()` / `ysize()` of subgraphs will carry through
- margins survive unchanged once `imargin()` is specified

This is one of the main reasons combine output often surprises users.

---

## 7. Exporting Graphs with `graph export`

### 7.1 Core syntax

```stata
graph export newfilename.suffix [, options]
```

By default, Stata determines the export format from the suffix:

| Suffix | Format |
|--------|--------|
| `.pdf` | PDF |
| `.eps` | EPS |
| `.svg` | SVG |
| `.png` | PNG |
| `.tif` | TIFF |
| `.jpg` | JPEG |
| `.gif` | GIF |
| `.ps` | PostScript |

You can also force the format with `as()`:

```stata
graph export mygraph, as(pdf)
```

### 7.2 Exporting the current graph

```stata
graph export figure.pdf, replace
```

This exports the graph currently displayed in the Graph window.

### 7.3 Exporting a graph stored on disk

```stata
graph use panel_main.gph
graph export panel_main.pdf, replace
```

Do not use `graph use, nodraw` if the next step is export.

### 7.4 Exporting a graph stored in memory

```stata
graph display panel_main
graph export panel_main.pdf, replace
```

Do not use `graph display, nodraw` if the next step is export.

### 7.5 `name()` and `replace`

```stata
graph export figure.eps, name(MyGraph) replace
```

Use `name()` when more than one graph window exists and you need to target a specific graph.

---

## 8. Export by Output Purpose

### 8.1 PDF: default publication vector format

Use PDF when:
- the output is for papers, drafts, appendices, or sharing
- you want scalable vector output
- the publisher accepts PDF directly

Typical command:

```stata
graph export figure.pdf, replace
```

### 8.2 EPS: legacy journal workflow

Use EPS when:
- the journal or publisher explicitly asks for EPS
- the downstream workflow is still PostScript-based

Typical command:

```stata
graph export figure.eps, replace
```

### 8.3 SVG: web and Illustrator-friendly vector output

Use SVG when:
- the graph is going to the web
- you need editable XML/vector output
- the workflow involves modern vector editing tools

Typical command:

```stata
graph export figure.svg, replace
```

### 8.4 PNG: Word, slides, and screen use

Use PNG when:
- the graph goes into PowerPoint, Word, or web slides
- a bitmap is acceptable or preferable
- you want explicit pixel dimensions

```stata
graph export figure.png, width(2400) replace
```

### 8.5 TIFF: bitmap for print requirements

Use TIFF when:
- the publisher requires TIFF
- a bitmap master file is requested for production

```stata
graph export figure.tif, width(2400) replace
```

---

## 9. Format-Specific Options

### 9.1 PDF options

Key PDF options:

| Option | Use |
|--------|-----|
| `width()` / `height()` | Physical size in inches |
| `mag()` | Magnification/shrinkage |
| `bgfill(off)` | Transparent background |
| `cmyk(on)` | CMYK colors instead of RGB |
| `fontface()` and friends | Override export fonts |

```stata
graph export figure.pdf, width(6) bgfill(off) replace
graph export figure.pdf, cmyk(on) replace
graph export figure.pdf, fontface(Calibri) replace
```

### 9.2 EPS options

Key EPS options:

| Option | Use |
|--------|-----|
| `cmyk(on)` | CMYK output when required |
| `mag()` | Magnification |
| `bgfill(off)` | Transparent background |
| `fontface()` family | PostScript font control |
| `orientation()` | Portrait vs landscape |

```stata
graph export figure.eps, cmyk(on) replace
graph export figure.eps, bgfill(off) replace
graph export figure.eps, fontface(Times) replace
```

The official manual recommends sticking to PostScript Core Font Set faces where possible.

### 9.3 PNG options

PNG is simple:

```stata
graph export figure.png, width(3000) height(2000) replace
```

Options:
- `width(#)` in pixels
- `height(#)` in pixels

If only one dimension is supplied, Stata computes the other from the aspect ratio.

### 9.4 TIFF options

TIFF is similar to PNG:

```stata
graph export figure.tif, width(3000) replace
```

Options:
- `width(#)` in pixels
- `height(#)` in pixels

### 9.5 SVG options

SVG has the richest export-specific controls:

| Option | Use |
|--------|-----|
| `width(#px | #in)` / `height(#px | #in)` | Pixel or inch output size |
| `bgfill(off)` | Transparent background |
| `ignorefont(on)` | Let renderer choose fonts |
| `pre(on)` | Preserve runs of spaces |
| `nbsp(on)` | Better browser compatibility for preserved spaces |
| `clipstroke(off)` | Smaller / simpler SVG files in some cases |
| `scalestrokewidth(on)` | Adobe Illustrator workaround |
| `pathprefix()` | Stable path names across files |

```stata
graph export figure.svg, width(1200px) bgfill(off) replace
graph export figure.svg, pre(on) replace
graph export figure.svg, scalestrokewidth(on) replace
```

---

## 10. Format Limitations and Risks

### 10.1 Fonts are format-dependent

PDF, EPS, and SVG all have format-specific font behavior.

Practical implications:
- exported text may not match the Graph window exactly
- some font overrides depend on OS and configuration
- EPS is safest with standard PostScript fonts

### 10.2 CMYK is for publisher requirements, not default use

If the journal requires CMYK, use PDF or EPS options that support it:

```stata
graph export figure.pdf, cmyk(on) replace
graph export figure.eps, cmyk(on) replace
```

For ordinary screen and document use, RGB is usually the right default.

### 10.3 Transparency and background handling vary

For vector formats, `bgfill(off)` is the main control for transparent backgrounds.

```stata
graph export figure.pdf, bgfill(off) replace
graph export figure.eps, bgfill(off) replace
graph export figure.svg, bgfill(off) replace
```

Always verify the result in the target application.

### 10.4 SVG renderer differences are real

The SVG section of the official manual makes this unusually explicit:
- `baselineshift()` has browser support differences
- `scalestrokewidth(on)` fixes Illustrator but harms other renderers
- `nbsp()` and `pre()` trade off readability, compactness, and compatibility

So SVG export should be tested in the actual downstream renderer.

### 10.5 `graph display` is not a free pass for redesign

The official guidance is cautious:
- redisplaying with a new scheme works best when the original graph stayed near default placements
- manually named colors and heavily customized placements may not migrate cleanly across schemes

---

## 11. Common Traps

### 11.1 Forgetting the `.gph` workflow

Do not jump straight from drawing to export when the graph may need revision or combination later. Save a live `.gph` first.

### 11.2 Confusing memory graphs with disk graphs

`name(g1)` stores in memory only.
`saving(g1)` or `graph save g1` creates a persistent `.gph`.

### 11.3 Expecting `graph combine` to preserve subgraph `xsize()` / `ysize()`

It does not. The manual explicitly says `graph combine` discards that physical-size information.

### 11.4 Using `imargin()` and then wondering where the original geometry went

`imargin()` overrides original margin information for subgraphs.

### 11.5 Applying `xcommon` / `ycommon` too broadly

They are for compatible previously drawn `twoway` graphs, not a universal alignment tool.

### 11.6 Exporting from the wrong graph window

If multiple graphs are open, use `name()` with `graph export`.

### 11.7 Using `nodraw` in a workflow that ends with export

The official export instructions explicitly say not to use `graph use, nodraw` or `graph display, nodraw` for export workflows.

---

## 12. Publication Templates

### 12.1 Reusable two-panel workflow

```stata
twoway scatter y1 x, name(g1)
twoway line y2 x, name(g2)
graph combine g1 g2, cols(2) iscale(1) ///
    saving("results/figures/combined_main", replace)
graph export "results/figures/combined_main.pdf", replace
```

Use when the final graph may need both revision and vector export.

### 12.2 Disk-based combine workflow

```stata
twoway scatter y1 x, saving("results/figures/panel_a", replace)
twoway line y2 x,    saving("results/figures/panel_b", replace)
graph combine "results/figures/panel_a" ///
              "results/figures/panel_b", ///
              cols(2) iscale(1)
graph export "results/figures/panels.pdf", replace
```

Use when panels are produced in separate scripts or stages.

### 12.3 Tight multi-panel economics figure

```stata
graph combine g1 g2 g3 g4, ///
    cols(2) ///
    ycommon xcommon ///
    imargin(0 0 0 0) ///
    iscale(*1.1) ///
    graphregion(margin(l=18 r=18))
graph export "figure_multi_panel.pdf", replace
```

Use when a four-panel figure must read as one coherent figure.

### 12.4 Slide / Word export

```stata
graph export "figure_slide.png", width(3000) replace
```

Use when the destination is not vector-native.

### 12.5 Publisher bitmap export

```stata
graph export "figure_print.tif", width(3000) replace
```

Use only when the submission system or publisher requests TIFF specifically.

---

## 13. Final Guidance

The safest Stata graphics workflow is:

1. Draw the graph.
2. Save a live `.gph`.
3. Redisplay or combine until the layout is right.
4. Export the final figure in the format required by the destination.

In other words: treat `.gph` as the working source, and treat PDF/PNG/EPS/SVG/TIFF as delivery formats.
