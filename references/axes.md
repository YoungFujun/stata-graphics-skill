# axes.md — 坐标轴完整控制

来源：g.pdf [G-3] `axis_choice_options`（pp.472-476）、`axis_label_options`（pp.477-493）、
`axis_options`（pp.494-495）、`axis_scale_options`（pp.496-503）、`axis_title_options`（pp.504-508）

---

## 选项族总览

坐标轴相关选项分为四族，均属于 **twoway-level 选项**（放在命令末尾，所有 plot 括号之外），
但 `yaxis()`/`xaxis()` 是例外——见第五节。

| 选项族 | 控制内容 |
|--------|---------|
| `axis_label_options` | 刻度值与标签（xlabel/ylabel 等） |
| `axis_scale_options` | 轴的缩放、范围、外观线条（xscale/yscale） |
| `axis_title_options` | 轴标题文字（xtitle/ytitle） |
| `axis_choice_options` | 多轴时指定 plot 使用哪条轴（yaxis/xaxis）—— **plot-level** |

`xline()`/`yline()` 参考线也是 twoway-level 选项，见第六节。

---

## 一、axis_label_options — 刻度与标签

### 1.1 完整语法

```stata
{y|x|t|z}{label|tick|mlabel|mtick}(rule_or_values [, suboptions])
```

| 选项 | 说明 |
|------|------|
| `ylabel()` / `xlabel()` | 主刻度：画刻度线 + 标签 |
| `ytick()` / `xtick()` | 主刻度：只画刻度线，不画标签 |
| `ymlabel()` / `xmlabel()` | 次刻度：画次刻度线 + 标签 |
| `ymtick()` / `xmtick()` | 次刻度：只画次刻度线 |

### 1.2 规则（rule）速查

| rule | 示例 | 说明 |
|------|------|------|
| `##` | `#6` | 让 Stata 选约 6 个"好看"的值 |
| `###` | `##10` | 在主刻度之间放 9 个次刻度（仅 mlabel/mtick 可用） |
| `#(#)#` | `-4(.5)3` | 从 -4 到 3，步长 0.5 |
| `minmax` | `minmax` | 只标最小值和最大值 |
| `none` | `none` | 不标任何值 |
| `.` | `.` | 跳过规则（等同于 none，但可与 add 连用） |

也可直接写 numlist：`ylabel(0 5 10 25 50)` 或混用规则与 numlist：`ylabel(0(5)50 75 100)`

### 1.3 字符串替换标签

```stata
// 用文字替换数值刻度
xlabel(1 "Jan" 2 "Feb" 3 "Mar" 4 "Apr" 5 "May" 6 "Jun" ///
       7 "Jul" 8 "Aug" 9 "Sep" 10 "Oct" 11 "Nov" 12 "Dec")

// Event study 时间轴（t=-1 为基准期）
xlabel(-4 "-4" -3 "-3" -2 "-2" -1 "Pre" 0 "Event" 1 "1" 2 "2" 3 "3")
```

### 1.4 关键 suboptions 完整表

| suboption | 说明 |
|-----------|------|
| `axis(#)` | 指定作用于第几条轴（多轴图时使用） |
| `add` | **追加**到已有刻度，而非替换；见陷阱 1 |
| `noticks` / `ticks` | 隐藏/强制显示刻度线 |
| `nolabels` / `labels` | 隐藏/强制显示标签文字 |
| `valuelabel` | 用变量的 value label 自动生成标签文字 |
| `format(%fmt)` | 格式化数值显示（如 `format(%9.2f)`、`format(%td)`） |
| `angle(anglestyle)` | 标签旋转角度：`angle(0)` 水平、`angle(90)` 垂直、`angle(45)` |
| `alternate` | 标签交错排列（解决 x 轴密集拥挤） |
| `norescale` | 新刻度值不触发轴范围自动扩展 |
| `labsize(textsizestyle)` | 标签字号（如 `labsize(small)`、`labsize(vsmall)`） |
| `labcolor(colorstyle)` | 标签颜色 |
| `labgap(size)` | 标签与刻度线之间的间距 |
| `labstyle(textstyle)` | 标签整体样式 |
| `labelminlen(#)` | 标签最小宽度（空格补齐，用于对齐多图的 y 轴） |
| `tlength(size)` | 刻度线长度 |
| `tposition(outside\|crossing\|inside)` | 刻度线方向（默认 outside） |
| `tlstyle()` / `tlwidth()` / `tlcolor()` | 刻度线样式/粗细/颜色 |
| `custom` | 只对当前 add 的标签应用渲染选项（不影响其他标签） |
| `grid` / `nogrid` | 在该轴的主刻度位置绘制/取消网格线 |
| `gmin` / `gmax` / `nogmin` / `nogmax` | 控制是否在最小/最大刻度处画网格线 |
| `gstyle(gridstyle)` | 网格线整体样式 |
| `gextend` / `nogextend` | 网格线是否延伸到 plot region 的边距 |
| `glstyle()` / `glwidth()` / `glcolor()` / `glpattern()` | 网格线具体属性 |
| `tstyle(tickstyle)` | 刻度线 + 标签整体样式 |

### 1.5 常用写法示例

```stata
// 基础：指定5个刻度
scatter y x, ylabel(#5)

// 精确范围：-0.2 到 0.4，步长 0.1
scatter y x, ylabel(-0.2(0.1)0.4)

// 格式化 + 角度（大数字）
scatter y x, ylabel(, format(%12.0gc) angle(0))

// x 轴标签交错（避免重叠）
scatter y x, xlabel(, alternate)

// 用 value label 自动生成标签
scatter y cat_var, xlabel(1 2 3, valuelabel)

// 追加单个特殊标签（不破坏默认标签）
scatter y x, xlabel(0, add) ylabel(0 "Zero", add)

// 仅修改格式，保留 Stata 自动选择的刻度值
scatter y x, xlabel(, format(%9.2f))

// 网格线（y 方向）
twoway line y x, ylabel(, grid glcolor(gs14) glpattern(solid))
```

---

## 二、axis_scale_options — 轴缩放与外观

### 2.1 语法

```stata
yscale(axis_suboptions)
xscale(axis_suboptions)
tscale(axis_suboptions)    // 时间轴（t 轴）
```

可简写：`ysc()` / `xsc()`，`range()` 可简写为 `r()`。

### 2.2 axis_suboptions 完整表

| suboption | 说明 |
|-----------|------|
| `axis(#)` | 指定作用于第几条轴 |
| `log` / `nolog` | 对数刻度（自然对数）；轴标签显示原始单位 |
| `reverse` / `noreverse` | 轴从大到小排列（倒序） |
| `range(numlist)` | 扩展轴范围；**只扩展不收窄**，见陷阱 2 |
| `off` / `on` | 完全隐藏/强制显示轴（含线、刻度、标签、标题） |
| `noline` / `line` | 隐藏/强制显示轴线本身（刻度、标签、标题保留） |
| `fill` | 配合 off 使用：隐藏轴线但保留空间 |
| `alt` | 把轴移到对面（y 轴→右侧，x 轴→顶部） |
| `fextend` | 轴线延伸穿过 plot region 及其边距（大多数 scheme 的默认） |
| `extend` | 轴线延伸穿过 plot region |
| `noextend` | 轴线不超出数据范围 |
| `titlegap(size)` | 轴标题与刻度标签之间的间距 |
| `outergap(size)` | 轴标题外侧的间距 |
| `lstyle(linestyle)` | 轴线整体样式 |
| `lcolor(colorstyle)` | 轴线颜色 |
| `lwidth(linewidthstyle)` | 轴线粗细 |
| `lpattern(linepatternstyle)` | 轴线图案（solid/dash/dot 等） |

### 2.3 常用写法

```stata
// 对数轴（标签显示原始值）
scatter y x, xscale(log)

// 对数轴（手动生成 log 变量后轴标签为 log 值，不推荐）
// 推荐：直接用 xscale(log)，Stata 自动处理标签

// 倒序 y 轴（如排名：1 在顶部）
scatter rank x, yscale(reverse)

// 确保 y 轴包含 0（不会收窄范围）
scatter y x, yscale(range(0))

// 完全隐藏 x 轴
scatter y x, xscale(off)

// 隐藏轴线但保留刻度和标签
scatter y x, yscale(noline)

// y 轴移到右侧
scatter y x, yscale(alt)

// 缩小刻度标签与轴线之间的间距
scatter y x, yscale(titlegap(1))
```

---

## 三、axis_title_options — 轴标题

### 3.1 语法

```stata
ytitle("string" ["string" ...] [, suboptions])
xtitle("string" ["string" ...] [, suboptions])
ttitle(...)    // 时间轴（ttitle 是 xtitle 的同义词）
```

`axis_title` 支持 Unicode 字符和 SMCL 标签（数学符号、斜体等）。

### 3.2 suboptions

| suboption | 说明 |
|-----------|------|
| `axis(#)` | 指定作用于第几条轴 |
| `prefix` | 将文字添加到现有标题之前 |
| `suffix` | 将文字添加到现有标题之后 |
| *textbox_options* | 控制文字外观（color、size、margin 等），见 [G-3] textbox_options |

### 3.3 常用写法

```stata
// 基础用法
scatter y x, ytitle("Coefficient") xtitle("Event time (quarters)")

// 多行标题（每行一个字符串）
scatter y x, ytitle("Estimated" "treatment effect")

// 留空标题（不显示轴标题）
scatter y x, ytitle("")

// 双轴图中第二条 y 轴单独命名
twoway (scatter gnp year, yaxis(1)) ///
       (scatter r year, yaxis(2)), ///
       ytitle("GNP (left)", axis(1)) ///
       ytitle("Interest rate (right)", axis(2))

// 隐藏双轴图第二条轴的标题
twoway ..., ytitle("", axis(2))

// SMCL 标签：希腊字母、上下标
scatter y x, ytitle("{&beta}{subscript:t}")
```

---

## 四、axis_choice_options — 多轴绑定（⚠ plot-level 选项）

### 4.1 关键规则

`yaxis()` 和 `xaxis()` 是 **plot-level 选项**，必须放在各 plot 的括号内，
**不能**放在 twoway 末尾。

```stata
// ❌ 错误：yaxis(2) 放在 twoway 层
twoway (scatter gnp year) (scatter r year), yaxis(2)

// ✅ 正确：yaxis(2) 放在第二个 plot 的括号内
twoway (scatter gnp year, yaxis(1)) (scatter r year, yaxis(2))
// 或简写（yaxis(1) 是默认值，可省略）
twoway (scatter gnp year) (scatter r year, yaxis(2))
```

### 4.2 语法

```stata
yaxis(# [#...])    // 1 ≤ # ≤ 9
xaxis(# [#...])    // 1 ≤ # ≤ 9
```

### 4.3 双 y 轴：两种用途

**用途 A：两个变量用不同刻度**

```stata
twoway (scatter gnp year) (scatter r year, yaxis(2)), ///
       ytitle("GNP", axis(1)) ytitle("Rate (%)", axis(2)) ///
       ylabel(#5, axis(1)) ylabel(#5, axis(2))
```

**用途 B：共享刻度但在两侧都显示（添加特殊标签）**

```stata
// yaxis(1 2)：左右两侧显示相同的刻度，可在右侧轴添加特殊标签
scatter bp concentration, yaxis(1 2) ylabel(120, axis(2) add)
```

### 4.4 各轴独立设置 xlabel/ylabel

```stata
// 为两条 y 轴分别设置标签
twoway (scatter gnp year) (scatter r year, yaxis(2)), ///
       ylabel(#5, axis(1))   ///
       ylabel(0 5 10 15, axis(2))
```

### 4.5 注意

- 每个 plot 至多有一条 x 轴和一条 y 轴（不能用 `yaxis(1 2)` 同时绑定到两条不同刻度的轴）
- 三条及以上 y 轴时，轴堆叠在同一侧（左侧），视觉上混乱，慎用

---

## 五、xline() / yline() — 参考线

`xline()` 和 `yline()` 是 **twoway-level 选项**（不属于 axis_* 族），
但与坐标轴密切相关，放置在命令末尾（plot 括号外）。

### 5.1 语法

```stata
yline(numlist [, suboptions])
xline(numlist [, suboptions])
```

| suboption | 说明 |
|-----------|------|
| `lstyle(linestyle)` | 整体线条样式 |
| `lcolor(colorstyle)` | 颜色 |
| `lwidth(linewidthstyle)` | 粗细 |
| `lpattern(linepatternstyle)` | 图案（solid/dash/dot/shortdash 等） |
| `axis(#)` | 指定作用于第几条轴（双轴图中区分） |

### 5.2 常用写法

```stata
// 零参考线（event study 标配）
twoway ..., yline(0, lcolor(gray) lpattern(dash))

// 处理时点参考线
twoway ..., xline(0, lcolor(black) lwidth(medthin) lpattern(solid))

// 同时画两条参考线
twoway ..., yline(0, lcolor(gray)) xline(0, lcolor(gray))

// 双 y 轴图中在第二条轴画参考线
twoway ..., yline(0, axis(2) lcolor(red) lpattern(dash))

// 多条参考线（numlist）
twoway ..., xline(-4 0 4, lcolor(gray) lpattern(shortdash))
```

---

## 六、高频陷阱速查

### 陷阱 1：`add` 缺失导致自定义标签替换默认标签

```stata
// 目的：在已有默认刻度基础上，额外标注 0 点
// ❌ 错误：直接写 ylabel(0) 会清空默认刻度，只剩 0 这一个标签
scatter y x, ylabel(0)

// ✅ 正确：加 add，追加到默认刻度
scatter y x, ylabel(0, add)

// ✅ 更常见：先指定主刻度规则，再追加特殊值
scatter y x, ylabel(#5) ylabel(0 "Ref", add custom labcolor(red))
```

### 陷阱 2：`range()` 只能扩展，不能收窄

```stata
// ❌ 错误期望：只显示 x 在 10 到 50 之间的数据
scatter y x, xscale(range(10 50))    // 不会截断数据，只扩展范围

// ✅ 正确：用 if 过滤数据
scatter y x if x >= 10 & x <= 50
```

### 陷阱 3：`yaxis()`/`xaxis()` 是 plot-level 选项

```stata
// ❌ 错误：放在 twoway 末尾
twoway (scatter y1 x) (scatter y2 x), yaxis(2)

// ✅ 正确：放在具体 plot 括号内
twoway (scatter y1 x) (scatter y2 x, yaxis(2))
```

### 陷阱 4：`##N` 次刻度的"减一"规则

```stata
// ##5 实际产生 4 个间隔刻度（Stata 将 5 理解为"等分数"，刻度数 = 5-1 = 4）
ymtick(##5)    // → 在每两个主刻度之间放 4 个次刻度

// ##10 实际产生 9 个次刻度（10-1=9）
xmtick(##10)   // → 9 个次刻度（接近"十分之一"间隔）

// 要精确控制，用 #(#)# 规则或 numlist 显式指定
```

### 陷阱 5：axis_options 归属 twoway 层而非 plottype 层

```stata
// ✅ axis_options 是 twoway 的选项，放在所有 plot 括号外
twoway (scatter y x) ///
       (line y2 x), ///
       ylabel(#5) xtitle("Time")    // ← 正确位置

// 可以混写在 plot 括号内（twoway 会拉取），但显式归到 twoway 层更规范
// yaxis()/xaxis() 是唯一必须在 plot 括号内的坐标轴选项
```

### 陷阱 6：`yscale(log)` 与手动生成 log 变量的区别

```stata
// yscale(log) 的优势：轴标签显示原始单位（如 1000, 10000, 100000）
scatter lexp gnppc, xscale(log)    // x 轴标签 = 原始 gnppc 值

// 手动 gen log_x = log(x) 再画图，标签显示的是 log 值（如 6.9, 9.2, 11.5）
// 读者需要心算换算回原始单位，通常不如 xscale(log) 直观
```

---

## 七、发表级常用配置模板

### Event study 图（完整坐标轴设置）

```stata
twoway (rcap hi lo time, lcolor(navy) lwidth(medium)) ///
       (scatter coef time, mcolor(navy) msize(medium) msymbol(circle)), ///
       yline(0, lcolor(gray) lpattern(dash) lwidth(thin)) ///
       xline(-1, lcolor(black) lwidth(thin) lpattern(shortdash)) ///
       xlabel(-6(1)6, labels) ///
       ylabel(-0.2(0.1)0.2, format(%9.2f) angle(0)) ///
       ytitle("Estimated coefficient") ///
       xtitle("Years relative to treatment") ///
       legend(off)
```

### 双 y 轴图（系数 + 密度）

```stata
twoway (scatter coef year, yaxis(1) mcolor(navy)) ///
       (kdensity pvalue, yaxis(2) lcolor(maroon) lpattern(dash)), ///
       ylabel(#5, axis(1)) ///
       ylabel(0(0.5)2, axis(2)) ///
       ytitle("Coefficient", axis(1)) ///
       ytitle("Density", axis(2)) ///
       xtitle("Year") ///
       yline(0, axis(1) lcolor(gray) lpattern(shortdash))
```

### 时间序列图（日期格式 x 轴）

```stata
twoway line y date, ///
       tlabel(01jan2010(1)01jan2020, format("%tdMon-YY") angle(45)) ///
       ytitle("Index") xtitle("") ///
       yscale(range(0))
```
