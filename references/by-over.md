# by-over.md — Choosing among `over()`, `by()`, and `graph combine`

Source: Stata 19 Graphics Reference Manual [G-3] `by_option` (pp.509-523);
[G-2] `graph combine` (pp.106-114); relevant `over()` sections in `graph bar`,
`graph box`, and `graph dot`; Mitchell (2022) §8.8 and Ch.4-Ch.6 examples.

---

## 1. Overview: Three Different Grouping Mechanisms

Stata has three different ways to show groups, subgroups, or multiple panels:

| Mechanism | What it does | Typical commands |
|-----------|--------------|------------------|
| `over()` | Builds categorical structure inside one graph | `graph bar`, `graph box`, `graph dot` |
| `by()` | Repeats the same graph command across groups and arranges the results into panels | `twoway`, `graph bar`, `graph box`, `graph dot`, many others |
| `graph combine` | Draws graphs separately, then assembles them later | any saved or named graphs |

These are not interchangeable.

**Practical decision rule:**
- Use `over()` when the grouping is part of one categorical display.
- Use `by()` when the same graph should be repeated across subgroups as small multiples.
- Use `graph combine` when the panels must be built separately or are not naturally one repeated command.

**Division of labor:**
- This file explains the choice among `over()`, `by()`, and `graph combine`.
- Detailed `graph combine` syntax belongs in [export-combine.md](./export-combine.md).
- Legend contents/location rules belong in [legend.md](./legend.md).
- Detailed bar/box/dot syntax belongs in [graph-bar-box.md](./graph-bar-box.md).

---

## 2. `over()` — Grouping Inside One Categorical Graph

### 2.1 What `over()` does

`over()` is not a panel system. It builds grouped categories inside a single graph.

```stata
graph hbar wage, over(occ)
graph box wage, over(sex) over(region)
graph dot wage, over(occupation, sort(1))
```

In official Stata syntax, `over()` belongs to the statistical graph commands such as:
- `graph bar`
- `graph hbar`
- `graph box`
- `graph hbox`
- `graph dot`

These commands summarize data by category and then display the summaries.

### 2.2 Why `over()` is usually preferred for bar/box/dot graphs

For explicitly categorical displays, `over()` is usually the default choice because:
- it keeps categories in one graph
- comparisons are immediate
- labels and spacing are designed for grouped summaries
- official documentation repeatedly treats `over()` as the natural structure for these commands

Mitchell and the official manuals both treat `by()` as secondary for bar/box/dot graphs: useful when the graph becomes too crowded, not as the default first step.

### 2.3 Multiple `over()` levels

```stata
graph hbar wage, over(occ5) over(collgrad)
graph box wage, over(industry) over(union)
graph dot wage, over(collgrad) over(occ5) over(urban2)
```

Think of repeated `over()` calls as nested categorical structure, not extra panels.

**Use `over()` when:**
- the graph is fundamentally one grouped comparison
- you want one shared axis and one visual field
- group labels belong on the same categorical axis
- readers should compare groups directly side-by-side

### 2.4 Limits and readability

`over()` is powerful, but not unlimited.

For `graph bar`, official limits depend on how many y variables are plotted:
- with one y variable, up to three `over()` variables
- with multiple y variables, up to two `over()` variables

Even before you hit a formal limit, the graph may become unreadable because:
- category labels get too long
- too many boxes or bars compress the plot
- subgroup hierarchy becomes visually confusing

When that happens, move to `by()` rather than trying to force more `over()` structure.

---

## 3. `by()` — Repeating One Graph Command across Groups

### 3.1 What `by()` does

`by()` takes one graph command and repeats it for each value of one or more grouping variables, then arrays the results in a panel layout.

```stata
twoway scatter wage age, by(sex)
graph dot wage, over(occ5) by(union)
graph hbox wage, over(industry) by(region)
```

Conceptually:
- one graph recipe
- multiple subsets
- one automatic panel arrangement

This is Stata's native small-multiples system.

### 3.2 When `by()` is the right tool

Use `by()` when:
- each panel is the same plot type and same design
- the reader should compare patterns across subgroups, not merge all groups into one axis
- the graph is too crowded with a single `over()` structure
- you want a coordinated panel layout without manually saving and combining graphs

This is especially common for:
- subgroup-specific trend or event-study plots
- repeated scatter or fit graphs
- bar/box/dot graphs that become too long with many categories

### 3.3 Core layout controls

The official `by_option` includes these important layout controls:

```stata
by(groupvar, rows(2) cols(3) colfirst holes(2) compact)
```

Key suboptions:

| Option | Role |
|--------|------|
| `rows(#)` / `cols(#)` | Set the panel matrix |
| `colfirst` | Fill by columns rather than rows |
| `holes(numlist)` | Leave selected cells blank |
| `compact` | Reduce extra space between panels |
| `imargin(marginstyle)` | Control panel margins |
| `total` | Add a panel for the full sample |
| `missing` | Include missing categories as their own panels |

`holes()` matters when the panel grid should follow a custom reading order or leave an intentional blank slot.

### 3.4 Scale behavior inside `by()`

One of the most important `by()` decisions is whether axes should remain comparable across panels.

Relevant official suboptions:

```stata
by(groupvar, yrescale)
by(groupvar, xrescale)
by(groupvar, rescale)
```

Meaning:

| Option | Meaning |
|--------|---------|
| default | common scales across panels where the graph type allows |
| `yrescale` | allow y scales to differ across panels |
| `xrescale` | allow x scales to differ across panels |
| `rescale` | allow both to differ |

**Recommendation:**
- Keep common scales when cross-panel level comparison matters.
- Use `yrescale` or `xrescale` only when panel-specific variation would otherwise be unreadable.
- Once scales differ, annotate clearly because visual comparisons become harder.

### 3.5 Titles and legends inside `by()`

`by()` changes where some options belong.

The official rule is:
- content-changing legend options go outside `by()`
- location-changing legend options go inside `by()`

```stata
twoway line y1 y2 x, ///
    legend(label(1 "Men") label(2 "Women")) ///
    by(group, legend(off))
```

Interpretation:
- `legend(label())` changes legend contents, so it stays outside
- `legend(off)` changes overall by-graph legend placement/display, so it goes inside

The same inside/outside distinction matters for titles:
- titles outside `by()` apply at the single-panel graph-command level
- titles inside `by()` apply to the overall panel array

See [legend.md](./legend.md) and [labels-text.md](./labels-text.md) for the detailed syntax.

---

## 4. `graph combine` — Assemble Separately Drawn Graphs

### 4.1 What makes `graph combine` different

`graph combine` does not repeat one command. It assembles graphs that already exist.

```stata
twoway scatter y x, name(g1)
twoway line y x, name(g2)
graph combine g1 g2, cols(2)
```

This means:
- each subgraph can have different data preparation
- each subgraph can have different graph types
- each subgraph can have different legends, scales, or annotations before assembly

### 4.2 When `graph combine` is the right tool

Use `graph combine` when:
- panels are not all generated by one command
- panels are different graph types
- graphs need separate creation pipelines
- subgraphs are reused across outputs
- you need manual control beyond what `by()` offers

Typical cases:
- a coefficient plot beside a histogram
- a main event-study panel beside a placebo panel
- graphs created in separate scripts or stages
- a heterogeneous figure built from stored `.gph` files

### 4.3 When not to use it

Do not default to `graph combine` when `by()` already solves the problem.

If all panels are:
- the same graph type
- built from one command
- sharing one design
- intended as small multiples

then `by()` is usually simpler, faster, and more coherent.

---

## 5. Decision Rules: Which One Should You Choose?

### 5.1 Fast routing table

| Goal | Best choice | Why |
|------|-------------|-----|
| Compare categories inside one bar/box/dot display | `over()` | Categorical structure belongs inside one graph |
| Repeat the same graph over subgroups | `by()` | Native small-multiples system |
| Assemble separately built or heterogeneous panels | `graph combine` | Post hoc composition |
| Too many categories for one bar/box/dot graph | usually `by()` | Keep the command, split the panels |
| Different plot types in one final figure | `graph combine` | `by()` cannot do heterogeneous panels |

### 5.2 A practical sequence

When you face grouped output, ask these questions in order:

1. Is this one categorical summary graph?
   If yes, start with `over()`.

2. Is it really the same graph repeated across groups?
   If yes, use `by()`.

3. Are the panels built differently or not naturally one command?
   If yes, use `graph combine`.

This sequence avoids a common mistake: jumping to `graph combine` too early.

---

## 6. Shared Issues across the Three Approaches

### 6.1 Legends

- `over()` often reduces the need for legends because categories may already be identified on the axis.
- `by()` may produce one overall legend rather than one legend per panel.
- `graph combine` inherits whatever each subgraph already contains unless those graphs were designed to suppress legends beforehand.

### 6.2 Titles and notes

- `over()` usually needs one graph-level title only.
- `by()` often works well with a shared title and automatic panel subtitles.
- `graph combine` usually needs the cleanest subgraphs possible, then one combined title/caption at the end.

### 6.3 Comparability of scales

- `over()` naturally shares one scale.
- `by()` can keep or relax common scales via `xrescale` / `yrescale`.
- `graph combine` requires active alignment choices such as `xcommon` / `ycommon`; see [export-combine.md](./export-combine.md).

### 6.4 Empty cells and missing categories

- `over()` may need `missing` or `allcategories` depending on the graph command.
- `by()` can include missing groups with `missing`.
- `graph combine` can leave blank cells intentionally with `holes()`.

These are not equivalent operations, so do not translate them mechanically.

---

## 7. Common Traps

1. Treating `over()` as if it were a panel command. It is not; it structures categories inside one graph.

2. Using `graph combine` for ordinary small multiples. If one command can generate all panels, `by()` is usually better.

3. Forgetting that `by()` changes option placement. Legend contents and legend location do not belong in the same place.

4. Letting `yrescale` or `xrescale` undermine comparability without warning the reader.

5. For bar/box/dot graphs, switching to `by()` too late. If category labels are unreadable, split the display earlier.

6. Assuming `graph combine` will automatically harmonize scales, margins, and legends. It will not.

7. Confusing repeated `over()` levels with repeated y variables. Those produce different visual structures.

---

## 8. Publication Templates

### 8.1 One grouped categorical display -> `over()`

```stata
graph hbar wage, ///
    over(industry, sort(1) descending) ///
    over(union) ///
    title("Mean wages by industry and union status") ///
    legend(off)
```

### 8.2 Same design repeated across subgroups -> `by()`

```stata
twoway ///
    (scatter wage age, mcolor(navy%60)) ///
    (lfit wage age, lcolor(navy)), ///
    by(sex, cols(2) compact note("") legend(off)) ///
    title("Wage-age profiles by sex")
```

### 8.3 Heterogeneous final figure -> `graph combine`

```stata
twoway scatter wage age, name(g1, replace)
graph hbar wage, over(education) name(g2, replace)

graph combine g1 g2, cols(2) ///
    title("Main descriptive patterns")
```

### 8.4 Too many categories for one graph -> `by()` over a grouped command

```stata
graph dot wage, ///
    over(occupation, sort(1)) ///
    by(region, cols(2) compact note("") legend(off))
```

This keeps the grouped categorical structure inside each panel while using `by()` to avoid an unreadably long single graph.
