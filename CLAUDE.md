# CLAUDE.md — stata-graphics-skill

## Project Overview

This project builds a **Stata graphics skill** for Claude Code: a structured set of reference
files that guide CC to look up syntax and options before writing any Stata graphics code,
rather than relying on training data alone.

Once installed, the skill lives in the Claude Code configuration directory and integrates
with the existing stata skill (see [dylantmoore/stata-skill](https://github.com/dylantmoore/stata-skill)).

---

## Installation Paths

| Platform | Claude config directory | Skill install path |
|----------|------------------------|-------------------|
| macOS / Linux | `~/.claude/` | `~/.claude/stata-graphics/` |
| Windows | `%USERPROFILE%\.claude\` | `%USERPROFILE%\.claude\stata-graphics\` |

After each session, copy completed reference files from `references/` in this repo to the
`stata-graphics/references/` folder in your Claude config directory.
The `SKILL.md` routing table goes directly into `stata-graphics/` (written last, Session G).

---

## Trigger Rule

Add the following to your global or project-level `CLAUDE.md` to activate the skill:

```
## Stata Graphics Skill Reference
When writing Stata graphics code—or when the task involves producing a figure, plot, or
visual output from Stata estimation results—read the stata-graphics SKILL.md for graphics
syntax, options reference, and routing to relevant files.

This applies to both explicit graph requests and implicit ones, including:
- Explicit: "画图"、"出图"、"可视化"、plot, figure, graph, chart, visualize
- Implicit commands: twoway, scatter, rcap, rarea, rspike, coefplot, marginsplot,
  event_plot, csdid_plot, graph bar, graph box, graph combine, graph export,
  kdensity, histogram
- Implicit task descriptions: "展示系数"、"系数图"、"平行趋势"、"event study 图"、
  "动态效应图"、"把结果可视化"、"论文图形"、"发表级图形"、
  "show coefficients", "parallel trends", "event study plot", "visualize results"
```

---

## Repository Structure

```
stata-graphics-skill/           ← this repo (development & documentation)
├── CLAUDE.md                   ← this file
├── PROGRESS.md                 ← session-by-session build log
├── README.md                   ← public-facing documentation
└── references/                 ← skill file drafts (copy to your Claude config after each session)
    ├── estimation-to-graph.md  ← bridge: regression results → plottable data
    ├── twoway-syntax.md
    ├── ci-bands.md
    ├── axes.md
    ├── schemes-regions.md
    ├── markers.md
    ├── lines.md
    ├── colors.md
    ├── labels-text.md
    ├── legend.md
    ├── graph-bar-box.md
    ├── export-combine.md
    └── by-over.md
```

---

## Reference Sources

1. **Mitchell (2022)** — *A Visual Guide to Stata Graphics*, 4th ed.
   Tutorial-style; teaches how to build graphs step by step. Available from Stata Press.

2. **Stata Graphics Reference Manual `[G]`** — bundled with every Stata installation.
   Official reference; authoritative for option syntax and valid style arguments.
   Organized as: [G-2] Commands · [G-3] Options · [G-4] Styles, concepts, and schemes.
   Access: type `help graph` in Stata, or find the PDF in your Stata `docs/` directory
   (macOS: `/Applications/Stata*/docs/g.pdf`; Windows: `C:\Program Files\Stata*\docs\g.pdf`).

3. **Existing stata skill** — `~/.claude/stata/` (or `%USERPROFILE%\.claude\stata\` on Windows).
   Primary source for Session A; covers regression commands and `e()` stored results.

---

## Session Build Order

| Session | Target Files | Source | Priority |
|---------|-------------|--------|----------|
| A | `estimation-to-graph.md` | existing stata skill (no PDF needed) | ★★★ |
| B | `twoway-syntax.md` + `ci-bands.md` | Mitchell Ch.2 + [G] manual | ★★★ |
| C | `axes.md` + `schemes-regions.md` | Mitchell Ch.8/9 + [G] manual | ★★ |
| D | `markers.md` + `lines.md` + `colors.md` | Mitchell Ch.10 + [G] manual | ★★ |
| E | `labels-text.md` + `legend.md` + `export-combine.md` | Mitchell Ch.8/11 + [G] manual | ★ |
| F | `graph-bar-box.md` + `by-over.md` | Mitchell Ch.4/5/6 + [G] manual | ★ |
| G | `SKILL.md` routing table | all above | final |

---

## Version Notes

Core `twoway`, `graph bar/box/combine/export` syntax is stable across Stata 14+.
No global version lock needed. Flag version-specific exceptions inline in the relevant file.
Reference materials are based on Stata 19 official manual and Mitchell (2022).

---

## Maintenance

**Passive (automatic):** When CC makes a graphics error and is corrected, trigger the
`learn` skill to record the fix in the relevant reference file.

**Active (template learning):** When a well-crafted figure with code is found (published
paper, replication archive, online example), prompt CC:
> "这是 [来源] 的代码，帮我学习提炼其中的关键技法，写入 graphics skill。"

**Version updates:** If a Stata upgrade introduces syntax changes, update the affected
reference file and note the version boundary inline.
