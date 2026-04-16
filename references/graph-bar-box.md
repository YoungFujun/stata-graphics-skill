# graph-bar-box.md — `graph bar`, `graph box`, and `graph dot`

Source: Stata 19 Graphics Reference Manual [G-2] `graph bar`, `graph box`,
and `graph dot`; Mitchell (2022) Ch.4-Ch.6.

---

## 1. Overview: Statistical Graph Commands, Not `twoway` Geometry

`graph bar`, `graph box`, and `graph dot` are not `twoway` plottypes.
They are higher-level statistical graph commands that:
- summarize data by category
- apply categorical layout rules automatically
- use `over()` as their main grouping syntax

That is why they often require less code than hand-built `twoway` graphs for descriptive summaries.

**Decision rule:**
- Use these commands for grouped descriptive statistics.
- Use `twoway` when you need low-level geometric control or custom overlays not supported by the statistical graph command.

**Division of labor:**
- Grouping logic across `over()`, `by()`, and `graph combine` belongs in [by-over.md](./by-over.md).
- Legend control belongs in [legend.md](./legend.md).
- Text and titles belong in [labels-text.md](./labels-text.md).

---

## 2. When to Use Which Command

| Command | Best use | Typical economics use |
|---------|----------|-----------------------|
| `graph bar` / `graph hbar` | Means, shares, counts, totals by category | descriptive subgroup means; composition by group |
| `graph box` / `graph hbox` | Distribution summary by category | wage distributions across groups; outcome dispersion by treatment status |
| `graph dot` | Clean multi-group mean comparison | many-category means where bars would look heavy |

**Practical rule:**
- Prefer `graph hbar` over vertical `graph bar` when category labels are long.
- Prefer `graph dot` over bar charts when the point estimate matters more than filled area.
- Prefer `graph box` when within-group spread is substantively important.

---

## 3. `graph bar` / `graph hbar`

### 3.1 Core idea

`graph bar` and `graph hbar` display summary statistics, not raw observations.

```stata
graph hbar wage, over(occupation)
graph bar (count), over(industry)
graph hbar (percent), over(region)
```

Official statistics include:

```stata
mean meanci median sum count percent min max
```

The main distinction is:
- `graph bar` is vertical
- `graph hbar` is horizontal

For publication work with many category labels, `graph hbar` is often easier to read.

### 3.2 Default statistic behavior

This is a common source of mistakes:

- if you supply a y variable, the default statistic is usually `mean`
- if you specify `over()` but no y variable, the default is `percent`

```stata
// Mean of wage by occupation
graph hbar wage, over(occ)

// Percent of observations by occupation
graph bar, over(occ)
```

Do not describe a default-percent graph as if it were plotting the variable itself.

### 3.3 `over()` hierarchy

Repeated `over()` calls build nested categories:

```stata
graph hbar wage, over(occ5) over(collgrad)
graph bar (meanci) wage, over(smsa) over(married) over(collgrad)
```

Important practical rules:
- each grouping variable gets its own `over()`
- repeated `over()` calls are not paneling
- too many levels can make the graph unreadable even if syntax still allows them

### 3.4 Sorting, gaps, and labels

Useful suboptions inside `over()`:

```stata
graph hbar wage, ///
    over(industry, sort(1) descending label(angle(0))) ///
    over(union, gap(20))
```

High-value controls:

| Control | Role |
|---------|------|
| `sort(1)` or `sort((stat) varname)` | order categories by a displayed statistic |
| `descending` / `reverse` | reverse the order |
| `label(...)` | style the category labels |
| `gap(#)` | widen space between over-groups |
| `relabel()` | replace category labels |

Do not confuse:
- `bargap()` -> spacing between bars for different y variables
- `over(..., gap())` -> spacing between grouped categories

### 3.5 Multiple y variables, `asyvars`, `stack`, and `percentages`

```stata
graph bar public private, over(country)
graph bar wage bonus, over(industry) asyvars
graph bar wage bonus, over(industry) stack percentages
```

Conceptually:
- multiple y variables create separate series
- `asyvars` treats those series more like separate categories
- `stack` stacks them rather than placing them side by side
- `percentages` expresses shares within the stack

This can be useful, but it is heavier to read than a clean `over()` structure.

### 3.6 `meanci`

`meanci` is useful when you want bar-like summaries with uncertainty attached:

```stata
graph bar (meanci) wage, over(smsa) over(married)
```

Remember:
- `meanci` is available in these statistical graph commands
- some layout options, such as `stack`, do not combine with it

---

## 4. `graph box` / `graph hbox`

### 4.1 What box plots summarize

A box plot summarizes a distribution rather than one central tendency only.

Officially, the display includes components such as:
- median
- quartiles
- adjacent values
- outside values

This makes `graph box` useful when group dispersion matters, not just group means.

```stata
graph box wage, over(sex)
graph hbox wage, over(industry, sort(1))
```

### 4.2 Grouping and ordering

`graph box` relies heavily on `over()`:

```stata
graph box wage, over(industry) over(union)
graph hbox wage, over(industry, sort(1)) nooutside
```

Use it when readers should compare group distributions within one graph.

Mitchell and the official manual both support this rule:
- prefer `over()` for box charts
- move to `by()` only when one graph becomes too long or crowded

### 4.3 Outliers and box-specific styling

Two especially useful controls are:

```stata
nooutside
marker(1, mcolor(navy) msymbol(o))
```

Use them when:
- outliers distract from the central pattern
- you want outside values visible but styled more lightly

`box(#, ...)` also allows box-specific styling for particular series in multiseries displays.

### 4.4 When `graph hbox` is better

Use `graph hbox` when:
- category names are long
- there are many groups
- the vertical layout would compress the boxes too tightly

Horizontal box plots are often more publication-friendly in economics tables-and-figures workflows.

---

## 5. `graph dot`

### 5.1 Why it matters

`graph dot` is often the cleanest descriptive graph in this family.

It uses the same summary-statistics logic as `graph bar`, but replaces filled bars with markers.

```stata
graph dot wage, over(occ, sort(1))
graph dot (meanci) wage, over(occ, sort(1))
```

This is often better than a bar chart when:
- the category count is high
- the comparison is about relative means, not area
- you want a lighter graph with less ink

### 5.2 Relationship to `graph hbar`

The official manual treats `graph hbar` and `graph dot` as closely related.

With multiple y variables:
- `graph hbar` draws multiple bars
- `graph dot` draws multiple markers on the same line

That can become cluttered, so Stata provides tools such as:

```stata
ascategory
linegap(#)
```

The official guidance is that `ascategory` is often the cleaner solution.

### 5.3 `over()` first, `by()` second

Normally, `graph dot` should still start with `over()`:

```stata
graph dot wage, over(collgrad) over(occ5)
```

Use `by()` when:
- you hit readability limits
- the graph would otherwise be too long
- you need one more grouping dimension than the display can comfortably absorb

---

## 6. Common Traps

1. Treating `graph bar` or `graph dot` as if they plot raw data. They plot summary statistics.

2. Forgetting that `graph bar, over(group)` with no y variable defaults to percentages of observations.

3. Confusing `bargap()` with `over(..., gap())`. They control different spacing problems.

4. Using too many `over()` levels just because syntax allows them. Readability fails before syntax does.

5. For box plots, defaulting to `by()` too early. Usually `over()` is the correct first design.

6. Using filled bars when `graph dot` would make the same comparison more clearly.

7. Forgetting that `allcategories` cannot be combined with `by()` in `graph bar`.

8. Using `stack` with `meanci`. The official command does not allow that combination.

---

## 7. Publication Templates

### 7.1 Horizontal means by category

```stata
graph hbar wage, ///
    over(industry, sort(1) descending label(angle(0))) ///
    title("Mean wages by industry") ///
    legend(off)
```

### 7.2 Grouped shares

```stata
graph bar (percent), ///
    over(occupation) ///
    over(female) ///
    title("Occupation shares by sex")
```

### 7.3 Distribution comparison with box plots

```stata
graph hbox wage, ///
    over(industry, sort(1)) ///
    over(union) ///
    nooutside ///
    title("Wage distributions by industry and union status")
```

### 7.4 Many-category comparison -> use dots, not bars

```stata
graph dot (meanci) wage, ///
    over(occupation, sort(1)) ///
    title("Mean wages with confidence intervals")
```

### 7.5 Too many categories for one graph -> split with `by()`

```stata
graph dot wage, ///
    over(occupation, sort(1)) ///
    by(region, cols(2) compact note("") legend(off))
```

This keeps the categorical summary logic of `graph dot` while using panels only to restore readability.
