# Changelog

## 2026-04-20

- Made graph templates scheme-first by removing hard-coded colors from most
  templates.
- Kept explicit transparent colors only where group overlap requires visual
  separation.
- Clarified `rcap` guidance so assistants treat both CI endpoints as bounds and
  do not replace either endpoint with the point estimate.
- Fixed the event-study `coefplot` template's baseline separator from
  `xline(2.5)` to `xline(4.5)`.
- Clarified that Claude Code paths in the README are examples and other AI
  assistants can use the same file layout in their own config directories.
- Reframed `/learn` as one way to automate a general template-learning workflow,
  not as a requirement for non-Claude tools.
