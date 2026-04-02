# PROGRESS.md — Build Log

Progress tracking for the stata-graphics-skill project.
Each session is independent; completed files are the persistent deliverables.

---

## Status Overview

| Session | Files | Status | Notes |
|---------|-------|--------|-------|
| A | `estimation-to-graph.md` | ⬜ Not started | No PDF needed; source: existing stata skill |
| B | `twoway-syntax.md`, `ci-bands.md` | ⬜ Not started | |
| C | `axes.md`, `schemes-regions.md` | ⬜ Not started | |
| D | `markers.md`, `lines.md`, `colors.md` | ⬜ Not started | |
| E | `labels-text.md`, `legend.md`, `export-combine.md` | ⬜ Not started | |
| F | `graph-bar-box.md`, `by-over.md` | ⬜ Not started | Low priority; start on demand |
| G | `SKILL.md` routing table | ⬜ Not started | Write after Sessions A–E complete |

Status codes: ⬜ Not started · 🔄 In progress · ✅ Complete · 🔁 Needs revision

---

## Session Logs

### Session A — estimation-to-graph.md
**Date:** —  
**Status:** ⬜ Not started  
**Objective:** Build the bridge file that converts regression stored results into plottable data.

Source files to read from existing stata skill:
- `linear-regression.md`
- `panel-data.md`
- `difference-in-differences.md`
- `regression-discontinuity.md`
- `reghdfe.md`
- `did.md`
- `event-study.md`
- `coefplot.md`
- `ivreg2.md`

> Note: Verify these files exist before starting; adjust scope based on actual coverage.

Planned deliverables:
- Stored results (`e()`) quick-reference table for major commands
- `matrix list e(b)` → `svmat` → reshape standard workflow
- `regsave` / `parmest` auxiliary extraction options
- Event study coefficient extraction: reghdfe path, csdid path, manual TWFE path
- `margins` → `marginsplot` vs. manual plot decision guide

**Log:** —

---

### Session B — twoway-syntax.md + ci-bands.md
**Date:** —  
**Status:** ⬜ Not started  
**Objective:** Core twoway syntax and all CI/error-band drawing methods.

Source: Mitchell Ch.2 + [G] manual: `graph twoway` (Commands), `twoway_options`,
`rcap_options`, `area_options` (Options).

Planned deliverables:
- All twoway plottypes quick reference (30+ types)
- Overlay syntax: `||` separator vs. `()`-binding notation
- Common overlay errors
- `rcap` / `rspike` / `rarea` / `rcapsym` comparison and selection guide
- Full event study code template: coefficients + CI + zero line + treatment timing line

**Log:** —

---

### Session C — axes.md + schemes-regions.md
**Date:** —  
**Status:** ⬜ Not started  
**Objective:** Complete axis control and overall graph style.

Source: Mitchell Ch.8/9 + [G] manual: `axis_options`, `axis_label_options`,
`axis_scale_options`, `region_options`, `Schemes intro`, scheme family descriptions.

Planned deliverables:
- `xlabel`/`ylabel` full syntax including custom string labels
- Reference lines (`xline`/`yline`) style control
- Dual y-axis binding syntax
- Scheme comparison and publication-level recommendation
- Graph size and margin conventions

**Log:** —

---

### Session D — markers.md + lines.md + colors.md
**Date:** —  
**Status:** ⬜ Not started  
**Objective:** Complete marker, line, and color option reference.

Source: Mitchell Ch.10 + [G] manual: `marker_options`, `line_options`, `colorstyle`,
`markerstyle`, `symbolstyle`, `linestyle`, `linepatternstyle`, `intensitystyle`.

Planned deliverables:
- Complete marker symbol table (`msymbol` valid arguments)
- `msize` units explained (absolute, relative, multiplier)
- Opacity syntax: `color%N` / `fcolor(%N)` (Stata 15+)
- Complete `lpattern` list
- Publication-ready color palette recommendations

**Log:** —

---

### Session E — labels-text.md + legend.md + export-combine.md
**Date:** —  
**Status:** ⬜ Not started  
**Objective:** Text annotation, legend control, and export/combination.

Source: Mitchell Ch.8/11 + [G] manual: `legend_options`, `title_options`,
`added_text_options`, `textbox_options`, `marker_label_options`,
`graph export`, `graph combine`, `saving_option`.

Planned deliverables:
- `legend()` complete option reference
- `graph combine` row/column layout and shared axis control
- `graph export` format and resolution specifications for journal submission

**Log:** —

---

### Session F — graph-bar-box.md + by-over.md
**Date:** —  
**Status:** ⬜ Not started  
**Priority:** Low; start on demand when bar/box/dot charts are needed.

Source: Mitchell Ch.4/5/6 + [G] manual: `graph bar`, `graph box`, `graph dot`, `by_option`.

**Log:** —

---

### Session G — SKILL.md routing table
**Date:** —  
**Status:** ⬜ Not started  
**Prerequisite:** Sessions A–E complete.

Planned deliverables:
- Task-type routing table: user description → which reference files to read
- High-frequency graphics pitfalls (twoway y/x variable order, overlay brackets,
  scheme conflicts, rcap upper/lower CI order, etc.)
- Integration notes with existing stata skill

**Log:** —

---

## Learned Templates

*This section records code patterns learned from published papers or high-quality examples,
extracted via the template learning mechanism.*

| Date | Source | Key techniques | Written to |
|------|--------|---------------|------------|
| — | — | — | — |

---

## Known Issues / Pending Fixes

*Log errors found during actual use that need to be incorporated into skill files.*

| Date | Error description | Root cause | Fixed in |
|------|------------------|------------|---------|
| — | — | — | — |
