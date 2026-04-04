# estimation-to-graph.md
# Bridge: Estimation Results → Plottable Data

When writing code that visualizes regression or causal inference results,
read this file BEFORE writing any extraction or reshaping code.

---

## 1. General e() Access

```stata
ereturn list               // Show all stored scalars, macros, matrices
matrix list e(b)           // Coefficient vector (1 × k)
matrix list e(V)           // Variance-covariance matrix (k × k)
display e(N)               // Sample size
display e(r2)              // R-squared (where applicable)
```

**Naming convention inside e(b):**
- OLS / reghdfe: column names are variable names, e.g. `e(b)[1,"mpg"]`
- Factor variables: `1.foreign`, `2.rep78`
- Interactions: `1.foreign#c.mpg`
- Event dummies (manual TWFE): `lead5`, `lead4`, ..., `lag0`, `lag1`, ...
- teffects: `ATE:r1vs0.treatment`, `POmean:0.treatment`
- csdid after `estat event`: `Tp0`, `Tp1`, ..., `Tm1`, `Tm2`, ...

---

## 2. e() Structures by Command Family

### 2.1 regress / reghdfe (OLS / HDFE)

| Stored result | Content |
|---------------|---------|
| `e(b)` | Coefficient vector |
| `e(V)` | VCV matrix |
| `e(N)` | Observations |
| `e(r2)` | R-squared |
| `e(r2_a)` | Adjusted R-squared |
| `e(rmse)` | Root MSE |
| `e(df_r)` | Residual degrees of freedom |
| `e(r2_within)` | Within R-squared *(reghdfe only)* |
| `e(df_a)` | Absorbed degrees of freedom *(reghdfe only)* |

Access individual coefficients and SEs:
```stata
display _b[varname]        // Coefficient
display _se[varname]       // Standard error
display _b[varname] / _se[varname]  // t-statistic
```

### 2.2 csdid (Callaway & Sant'Anna)

After `csdid ..., ivar() time() gvar()`:
```stata
estat event, window(-5 10) estore(cs_event)
// e(b) contains Tp0, Tp1, ..., Tm1, Tm2, ... stubs
// Use event_plot with stub_lag(Tp#) stub_lead(Tm#)
```

After `estat simple` / `estat group` / `estat calendar`:
```stata
display e(b)[1,1]          // ATT point estimate
display sqrt(e(V)[1,1])    // SE
```

### 2.3 did_multiplegt (de Chaisemartin & D'Haultfoeuille)

Stores results as named scalars, NOT as e(b)/e(V):

| Stored result | Content |
|---------------|---------|
| `e(effect)` | Effect at t=0 |
| `e(se_effect)` | SE of effect at t=0 |
| `e(dynamic_k)` | Dynamic effect at post-period k |
| `e(se_dynamic_k)` | SE of dynamic effect k |
| `e(placebo_k)` | Placebo estimate at pre-period -k |
| `e(se_placebo_k)` | SE of placebo k |
| `e(lower_bound)` | Matrix of lower CI bounds *(with `save_results`)* |
| `e(upper_bound)` | Matrix of upper CI bounds |
| `e(p_jointplacebo)` | p-value for joint placebo test |

> **Key difference:** Cannot use `_b[varname]` or event_plot directly.
> Must extract via matrix or scalar loops (see Section 3.3).

### 2.4 rdrobust

| Stored result | Content |
|---------------|---------|
| `e(tau_bc)` | Bias-corrected RD estimate |
| `e(tau_cl)` | Conventional RD estimate |
| `e(se_tau_bc)` | SE (robust, bias-corrected) |
| `e(h_l)`, `e(h_r)` | Bandwidth left/right |
| `e(N_h_l)`, `e(N_h_r)` | Effective N within bandwidth |

> **Visualization:** Use `rdplot` (separate command), not twoway manually.
> See graph-templates.md for rdplot template.

### 2.5 teffects (IPW / RA / AIPW / PSMatch)

`e(b)` uses a multi-equation naming convention:

```stata
teffects aipw (outcome covs) (treatment covs), ate pomeans
// e(b) columns: ATE:r1vs0.treatment, POmean:0.treatment, POmean:1.treatment
display _b[ATE:r1vs0.treatment]    // ATE point estimate
display _se[ATE:r1vs0.treatment]   // SE
```

For heterogeneity across subgroups, loop and store manually (see Section 3.2).

### 2.6 ivreg2 / ivregress

Same structure as `regress`: `e(b)`, `e(V)`, `_b[varname]`, `_se[varname]`.

---

## 3. Extraction Workflows

### 3.1 Direct scalar extraction: _b[] / _se[]

**Use when:** reghdfe / regress event study with manually created dummies.

```stata
// After reghdfe with dummies lead5 lead4 lead3 lead2 lag0 lag1 ... lag8
// (lead1 = 0 baseline, omitted)

preserve
clear
local n_pre  = 5   // number of pre-treatment periods (excluding baseline)
local n_post = 9   // number of post-treatment periods (0 through 8)
set obs `= `n_pre' + 1 + `n_post''

gen period   = .
gen b        = .
gen se       = .

local row = 1
forvalues k = `n_pre'(-1)2 {
    replace period = -`k' in `row'
    replace b      = _b[lead`k']  in `row'
    replace se     = _se[lead`k'] in `row'
    local row = `row' + 1
}
// Baseline period (-1): coefficient = 0 by normalization
replace period = -1 in `row'
replace b      = 0  in `row'
replace se     = 0  in `row'
local row = `row' + 1

forvalues k = 0/`n_post' {
    replace period = `k'          in `row'
    replace b      = _b[lag`k']   in `row'
    replace se     = _se[lag`k']  in `row'
    local row = `row' + 1
}

gen ci_lower = b - 1.96 * se
gen ci_upper = b + 1.96 * se
// → proceed to twoway (scatter b period)(rcap ci_lower ci_upper period)
restore
```

### 3.2 Matrix → svmat workflow

**Use when:** did_multiplegt (stores results as matrices after `robust_dynamic`).

```stata
// After did_multiplegt ... robust_dynamic dynamic(5) placebo(3)
preserve
clear
local n_pre  = 3
local n_post = 5
set obs `= `n_pre' + 1 + `n_post''

gen period   = .
gen estimate = .
gen se       = .

// Pre-treatment (placebos)
forvalues k = 1/`n_pre' {
    local row = `n_pre' - `k' + 1
    replace period   = -`k'                in `row'
    replace estimate = e(placebo_`k')      in `row'
    replace se       = e(se_placebo_`k')   in `row'
}
// t = 0
local row0 = `n_pre' + 1
replace period   = 0            in `row0'
replace estimate = e(effect)    in `row0'
replace se       = e(se_effect) in `row0'
// Post-treatment
forvalues k = 1/`n_post' {
    local row = `n_pre' + 1 + `k'
    replace period   = `k'                  in `row'
    replace estimate = e(dynamic_`k')       in `row'
    replace se       = e(se_dynamic_`k')    in `row'
}

gen ci_lower = estimate - 1.96 * se
gen ci_upper = estimate + 1.96 * se
restore
```

**Alternative when `e(lower_bound)` / `e(upper_bound)` matrices are available:**
```stata
preserve
clear
matrix effects = e(effect)
matrix lb      = e(lower_bound)
matrix ub      = e(upper_bound)
svmat effects, name(effects)
svmat lb,      name(lb)
svmat ub,      name(ub)
rename effects1 estimate
rename lb1      ci_lower
rename ub1      ci_upper
gen period = _n - `n_pre' - 1   // adjust offset for your pre-period count
restore
```

### 3.3 regsave / parmest (auxiliary packages)

**regsave** — saves `e(b)`, `e(V)`, t-stats, p-values to a .dta:
```stata
ssc install regsave
reghdfe outcome lead5-lead2 lag0-lag8, absorb(unit_id year) vce(cluster unit_id)
regsave using "event_coefs.dta", t p replace
// Variables in saved file: var, coef, stderr, t, p, N

use "event_coefs.dta", clear
keep if regexm(var, "^lead|^lag")
gen period = real(subinstr(subinstr(var,"lead","-",.),"lag","",.)
// → reshape into period/coef/ci format manually
```

**parmest** — saves full set of estimates including CI bounds:
```stata
ssc install parmest
reghdfe outcome ..., absorb(...) vce(cluster unit_id)
parmest, saving("event_parmest.dta", replace)
// Variables: parm, estimate, stderr, t, p, min95, max95
use "event_parmest.dta", clear
keep if regexm(parm, "^lead|^lag")
```

> **When to use:** regsave / parmest are most useful when saving results
> from multiple models for later comparison, or when you need p-values
> alongside CIs in the plotting dataset.

---

## 4. margins → Plot Decision Guide

| Situation | Recommended approach |
|-----------|---------------------|
| Nonlinear model (logit, probit, tobit, Poisson) | `margins, dydx(*)`  then `marginsplot` |
| Continuous interaction: effect of X at different values of Z | `margins, dydx(X) at(Z=(v1 v2 v3))` then `marginsplot` |
| Predicted probabilities over a range | `margins, at(X=(min(1)max))` then `marginsplot` |
| Custom styling needed / combining with other estimates | `margins, post` → extract `_b[]/_se[]` → hand-code `twoway` |
| Linear model, main effects only | Skip `margins`; plot `_b[var]` ± 1.96×`_se[var]` directly |

**Gotcha:** `margins, post` replaces `e()` with the margins results. After
`margins, post`, you CANNOT run `estat`, `predict`, or another `margins`
call on the original model. Only use `post` when you need to run `test` or
`lincom` on the margins themselves, or when extracting for hand-coded plots.

**Basic marginsplot template:**
```stata
regress y c.x1##c.x2 controls
margins, dydx(x1) at(x2=(10(10)60))
marginsplot, ///
    xtitle("Value of X2") ytitle("Marginal Effect of X1") ///
    yline(0, lcolor(black) lpattern(dash)) ///
    graphregion(color(white))
```
