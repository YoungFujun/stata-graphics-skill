# Changelog

## 2026-04-20

### 中文

- 将图形模板改为 scheme-first：默认让 Stata scheme 管理颜色，避免模板与
  `SKILL.md` 的配色原则冲突。
- 删除多数模板中的硬编码颜色，包括 `mcolor()`、`lcolor()`、`color()`、
  `bgcolor()` 和 `graphregion(color())`。
- 只在需要组间区分的重叠图中保留显式透明颜色，并在模板注释中说明原因。
- 完善 `rcap` 说明，引导 AI 助手把两个置信区间端点都视为区间边界，不要
  用点估计替代其中任何一个端点。
- 修正 event-study `coefplot` 模板中的基准线位置：从 `xline(2.5)` 改为
  `xline(4.5)`，与 4 个 pre-period coefficients 的说明一致。
- 补充 README 中的跨助手安装说明，明确 `~/.claude/` 是 Claude Code 示例
  路径，其他 AI 助手可使用自己的配置目录并保持相同文件结构。
- 将 `/learn` 改写为通用模板学习流程的一种自动化方式，而不是非 Claude
  工具必须支持的命令。

### English

- Made graph templates scheme-first so Stata schemes manage colors by default,
  avoiding conflicts with the color guidance in `SKILL.md`.
- Removed hard-coded colors from most templates, including `mcolor()`,
  `lcolor()`, `color()`, `bgcolor()`, and `graphregion(color())`.
- Kept explicit transparent colors only where overlap plots require group
  separation, with comments explaining the exception.
- Clarified `rcap` guidance so AI assistants treat both CI variables as interval
  endpoints and do not replace either endpoint with the point estimate.
- Fixed the event-study `coefplot` baseline separator from `xline(2.5)` to
  `xline(4.5)`, matching the note for four pre-period coefficients.
- Clarified in the README that `~/.claude/` is an example Claude Code path and
  other AI assistants can use their own config directories with the same file
  layout.
- Reframed `/learn` as one way to automate a general template-learning workflow,
  not as a command required by non-Claude tools.
