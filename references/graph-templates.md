# graph-templates.md
# Ready-to-Run Stata Graphics Templates

This file contains complete, copy-paste-ready code templates for common
figure types in empirical economics. Each template is annotated with key
options that are non-obvious or frequently mis-specified.

**To add a new template from a real-world example:** use `/learn` and direct
it to append to this file, following the format below.

---

## Template Format

```
## [Figure Type — Method/Style]
<!-- Source: [paper/package/example — optional] -->
<!-- Scenario: [one-line description of when to use] -->
<!-- Key technique: [non-obvious options or patterns] -->
[code block]
```

---

## 1. Event Study

### 1A. Event Study — Manual twoway (hand-coded)

<!-- Source: event-study.md, did.md from dylantmoore/stata-skill -->
<!-- Scenario: Any event study after extracting coefs into period/b/ci_lower/ci_upper dataset -->
<!-- Key technique: rcap takes (ci_lower ci_upper period) NOT (b ci period); xline at -0.5 not 0 to fall between bars -->

```stata
* Assumes dataset with variables: period, b (or estimate), ci_lower, ci_upper
* period = event time relative to treatment (negative = pre, 0+ = post)
* baseline period (-1) should have b=0, ci_lower=0, ci_upper=0

twoway ///
    (scatter b period, mcolor(navy) msymbol(circle) msize(medium)) ///
    (rcap ci_lower ci_upper period, lcolor(navy) lwidth(medium)), ///
    xline(-0.5, lcolor(cranberry) lpattern(dash) lwidth(medium)) ///
    yline(0, lcolor(black) lwidth(thin)) ///
    xlabel(-5(1)8) ///
    xtitle("Years Relative to Treatment") ///
    ytitle("Treatment Effect") ///
    legend(off) ///
    graphregion(color(white)) bgcolor(white)
```

**Syntax note:** `rcap ci_lower ci_upper period` — the CI bounds come BEFORE
the x-variable. Reversed order produces no error but wrong output.

---

### 1B. Event Study — coefplot (from reghdfe with named dummies)

<!-- Source: event-study.md from dylantmoore/stata-skill -->
<!-- Scenario: reghdfe with manually created lead/lag dummies; no separate dataset needed -->
<!-- Key technique: coeflabels() renames dummies to readable labels; xline at k+0.5 where k = position of last pre-period dummy -->

```stata
* Assumes: reghdfe outcome lead5 lead4 lead3 lead2 lag0-lag8, absorb(...) vce(cluster ...)
* (lead1 = 0, omitted as baseline)

coefplot, ///
    keep(lead5 lead4 lead3 lead2 lag0 lag1 lag2 lag3 lag4 lag5 lag6 lag7 lag8) ///
    vertical ///
    yline(0, lcolor(black) lwidth(thin)) ///
    xline(2.5, lcolor(cranberry) lpattern(dash) lwidth(medium)) ///
    coeflabels( ///
        lead5="-5" lead4="-4" lead3="-3" lead2="-2" ///
        lag0="0" lag1="1" lag2="2" lag3="3" lag4="4" ///
        lag5="5" lag6="6" lag7="7" lag8="8") ///
    xtitle("Periods Relative to Treatment") ///
    ytitle("Treatment Effect") ///
    graphregion(color(white)) bgcolor(white)
```

**Note on xline position:** with 4 pre-period dummies (lead5–lead2) shown
left to right, the baseline gap falls between position 4 and 5, so
`xline(4.5)`. Count how many pre-period coefficients you keep, then use
`xline(count + 0.5)`.

---

### 1C. Event Study — event_plot (publication quality)

<!-- Source: event-study.md from dylantmoore/stata-skill -->
<!-- Scenario: reghdfe-based event study; requires ssc install event_plot -->
<!-- Key technique: stub_lag(lag#) stub_lead(lead#) uses # as wildcard; scatter_opt and rcap_opt control point/CI style separately -->

```stata
* After: reghdfe outcome lead5-lead2 lag0-lag8, absorb(...) vce(cluster ...)
estimates store es_results

event_plot es_results, ///
    stub_lag(lag#) stub_lead(lead#) ///
    plottype(scatter) ciplottype(rcap) together ///
    graph_opt( ///
        xtitle("Years Relative to Policy", size(medlarge)) ///
        ytitle("Effect (Percentage Points)", size(medlarge)) ///
        xlabel(-5(1)8, labsize(medsmall)) ///
        ylabel(, labsize(medsmall) format(%4.2f) angle(0)) ///
        xline(-0.5, lcolor(cranberry) lpattern(dash) lwidth(medium)) ///
        yline(0, lcolor(black) lwidth(thin)) ///
        graphregion(color(white) margin(medium)) ///
        plotregion(margin(small)) bgcolor(white) ///
        title("") legend(off)) ///
    scatter_opt(mcolor(navy) msize(large) msymbol(circle)) ///
    rcap_opt(lcolor(navy) lwidth(medthick))

graph export "event_study.png", replace width(3000)
graph export "event_study.eps", replace
```

**CI style alternatives:**
```stata
ciplottype(rarea) ... ciopt(color(navy%20))      // shaded area (modern)
ciplottype(rcap)                                  // range caps (traditional)
plottype(line) ciplottype(rarea) ciopt(color(navy%15))  // line + shaded area
```

---

### 1D. Event Study — csdid (Callaway & Sant'Anna)

<!-- Source: did.md from dylantmoore/stata-skill -->
<!-- Scenario: staggered DiD using csdid package; requires ssc install csdid drdid -->
<!-- Key technique: must run estat event before plotting; estore() name is passed to event_plot -->

```stata
csdid outcome controls, ///
    ivar(unit_id) time(year) gvar(treatment_year) ///
    method(dripw) notyet

estat event, window(-5 10) estore(cs_event)
estat pretrend   // check parallel trends: p > 0.10 is good

event_plot cs_event, ///
    stub_lag(Tp#) stub_lead(Tm#) together ///
    plottype(scatter) ciplottype(rcap) ///
    graph_opt( ///
        xtitle("Periods to Treatment") ///
        ytitle("Average Treatment Effect") ///
        xlabel(-5(1)10) ///
        xline(-0.5, lcolor(cranberry) lpattern(dash)) ///
        yline(0, lcolor(black)) ///
        graphregion(color(white)) legend(off))
```

---

### 1E. Event Study — did_multiplegt (manual extraction)

<!-- Source: did.md from dylantmoore/stata-skill -->
<!-- Scenario: did_multiplegt with robust_dynamic; results are stored as scalars not e(b), so event_plot cannot be used directly -->
<!-- Key technique: loop over e(placebo_k) and e(dynamic_k) to build dataset manually -->

```stata
did_multiplegt outcome unit_id year treated, ///
    robust_dynamic dynamic(5) placebo(3) ///
    breps(100) cluster(unit_id)

local n_pre  = 3
local n_post = 5

preserve
clear
set obs `= `n_pre' + 1 + `n_post''
gen period   = .
gen estimate = .
gen se       = .

forvalues k = 1/`n_pre' {
    local row = `n_pre' - `k' + 1
    replace period   = -`k'              in `row'
    replace estimate = e(placebo_`k')    in `row'
    replace se       = e(se_placebo_`k') in `row'
}
local row0 = `n_pre' + 1
replace period   = 0            in `row0'
replace estimate = e(effect)    in `row0'
replace se       = e(se_effect) in `row0'
forvalues k = 1/`n_post' {
    local row = `n_pre' + 1 + `k'
    replace period   = `k'               in `row'
    replace estimate = e(dynamic_`k')    in `row'
    replace se       = e(se_dynamic_`k') in `row'
}

gen ci_lower = estimate - 1.96 * se
gen ci_upper = estimate + 1.96 * se

twoway ///
    (scatter estimate period, mcolor(navy) msymbol(circle)) ///
    (rcap ci_lower ci_upper period, lcolor(navy)), ///
    xline(-0.5, lcolor(cranberry) lpattern(dash)) ///
    yline(0, lcolor(black)) ///
    xlabel(-3(1)5) ///
    xtitle("Periods Relative to Treatment") ///
    ytitle("Treatment Effect") ///
    legend(off) graphregion(color(white))
restore
```

---

### 1F. Event Study — Multi-estimator comparison

<!-- Source: event-study.md from dylantmoore/stata-skill -->
<!-- Scenario: comparing TWFE, CS, Sun-Abraham on same figure; requires event_plot -->
<!-- Key technique: perturb() offsets dots horizontally so they don't overlap; each estimator has its own stub_lag/stub_lead pair -->

```stata
* Assumes: twfe, cs_event, sa each stored via estimates store / estore

event_plot twfe cs_event sa, ///
    stub_lag(lag#   Tp#  treat_#_) ///
    stub_lead(lead# Tm#  treat_#_) ///
    plottype(scatter) ciplottype(rcap) ///
    together perturb(-0.25(0.25)0.25) ///
    graph_opt( ///
        xtitle("Periods to Treatment") ///
        ytitle("Treatment Effect") ///
        xlabel(-8(2)8) ///
        xline(-0.5, lcolor(cranberry) lpattern(dash)) ///
        yline(0, lcolor(black)) ///
        graphregion(color(white)) ///
        legend(order(1 "TWFE" 3 "Callaway-Sant'Anna" 5 "Sun-Abraham") rows(1)))

graph export "method_comparison.png", replace width(3500)
```

---

## 2. Coefficient Plots

### 2A. Single model coefficient plot (horizontal, publication quality)

<!-- Source: coefplot.md from dylantmoore/stata-skill -->
<!-- Scenario: display OLS/IV/probit coefficients with CIs; drop _cons by default -->
<!-- Key technique: coeflabels() adds readable names; xline(0) adds null reference -->

```stata
* After any estimation command that stores e(b)/e(V)

coefplot, ///
    drop(_cons) ///
    xline(0, lcolor(black) lwidth(thin) lpattern(dash)) ///
    mcolor(navy) msymbol(circle) msize(medium) ///
    ciopts(lwidth(medium) lcolor(navy)) levels(95) ///
    coeflabels( ///
        mpg    = "Miles per Gallon" ///
        weight = "Vehicle Weight" ///
        foreign = "Foreign Origin") ///
    xtitle("Coefficient (95% CI)") ///
    graphregion(color(white)) bgcolor(white)

graph export "coefplot.pdf", replace
```

---

### 2B. Multi-model coefficient comparison

<!-- Source: coefplot.md from dylantmoore/stata-skill -->
<!-- Scenario: same outcome, multiple specifications side by side -->
<!-- Key technique: each model in parentheses with label(); offset() staggers confidence intervals -->

```stata
estimates store m1
* (run model 2)
estimates store m2
* (run model 3)
estimates store m3

coefplot (m1, label("No Controls")) ///
         (m2, label("With Controls")) ///
         (m3, label("Controls + FE")), ///
    drop(_cons) ///
    xline(0, lcolor(black) lwidth(thin) lpattern(dash)) ///
    mcolor(navy maroon forest_green) ///
    msymbol(circle square diamond) ///
    ciopts(lwidth(medium)) levels(95) ///
    legend(rows(1) position(6)) ///
    xtitle("Coefficient (95% CI)") ///
    graphregion(color(white)) bgcolor(white)

graph export "model_comparison.pdf", replace
```

---

### 2C. Vertical coefficient plot (many models / subgroup heterogeneity)

<!-- Scenario: comparing many models or subgroups on y-axis, coefficient on x-axis -->
<!-- Key technique: vertical flips axes; yline(0) replaces xline(0) -->

```stata
coefplot m1 m2 m3 m4 m5, ///
    drop(_cons) ///
    vertical ///
    yline(0, lcolor(black) lwidth(thin) lpattern(dash)) ///
    mcolor(navy) msymbol(circle) ///
    ytitle("Coefficient (95% CI)") ///
    graphregion(color(white)) bgcolor(white)
```

---

## 3. Marginal Effects / Predicted Values

### 3A. Marginsplot (default, nonlinear models)

<!-- Scenario: logit/probit/tobit — after margins over a continuous variable range -->
<!-- Key technique: marginsplot uses whatever was stored by margins; yline(0) useful for dydx plots -->

```stata
logit union i.south c.age##c.grade
margins, dydx(age) at(grade=(8(2)20))

marginsplot, ///
    xtitle("Years of Education") ///
    ytitle("Marginal Effect of Age on Union Membership") ///
    yline(0, lcolor(black) lwidth(thin) lpattern(dash)) ///
    graphregion(color(white)) bgcolor(white) ///
    plot1opts(lcolor(navy) lwidth(medthick)) ///
    ci1opts(color(navy%30))
```

---

### 3B. Predicted probabilities plot (manual twoway)

<!-- Scenario: need custom styling or combining with other plot elements -->
<!-- Key technique: margins, post replaces e() — do not run margins, post unless you are done with other post-estimation -->

```stata
logit union i.south c.age##c.grade
margins, at(age=(20(5)60)) post
* e() now contains marginal predictions

* Extract and plot manually
matrix b = e(b)
matrix V = e(V)
local k = colsof(b)
preserve
clear
set obs `k'
gen age = 20 + (_n-1)*5
gen pred = .
gen ci_lower = .
gen ci_upper = .
forvalues i = 1/`k' {
    replace pred     = b[1,`i']                    in `i'
    replace ci_lower = b[1,`i'] - 1.96*sqrt(V[`i',`i']) in `i'
    replace ci_upper = b[1,`i'] + 1.96*sqrt(V[`i',`i']) in `i'
}

twoway ///
    (rarea ci_lower ci_upper age, color(navy%20)) ///
    (line pred age, lcolor(navy) lwidth(medthick)), ///
    xlabel(20(10)60) ///
    xtitle("Age") ///
    ytitle("Predicted Probability of Union Membership") ///
    legend(off) graphregion(color(white))
restore
```

---

## 4. Regression Discontinuity

### 4A. RD plot (rdplot — standard)

<!-- Source: rdrobust.md from dylantmoore/stata-skill -->
<!-- Scenario: visualize the RD jump; use rdplot not manual scatter -->
<!-- Key technique: h() from rdrobust constrains the local polynomial fit to the bandwidth; ci(95) shade adds CI band -->

```stata
rdrobust outcome running_var
local h_opt = e(h_l)   // store optimal bandwidth

rdplot outcome running_var, ///
    h(`h_opt') kernel(triangular) ///
    ci(95) shade ///
    nbins(20) p(4) ///
    graph_options( ///
        title("") ///
        xtitle("Running Variable (centered at cutoff)") ///
        ytitle("Outcome") ///
        xline(0, lcolor(cranberry) lpattern(dash) lwidth(medium)) ///
        graphregion(color(white)) bgcolor(white) legend(off))

graph export "rd_plot.png", replace width(2400) height(1800)
graph export "rd_plot.eps", replace
```

---

## 5. Propensity Score / Balance

### 5A. Propensity score overlap (histogram, two groups)

<!-- Source: treatment-effects.md from dylantmoore/stata-skill -->
<!-- Scenario: checking common support after PSM or IPW estimation -->
<!-- Key technique: color(%30) makes both histograms semi-transparent for overlap visibility -->

```stata
* Assumes: ps = propensity score variable, treatment = binary indicator

twoway ///
    (histogram ps if treatment==0, color(blue%30) width(0.05)) ///
    (histogram ps if treatment==1, color(red%30) width(0.05)), ///
    xlabel(0(0.1)1) ///
    xtitle("Propensity Score") ///
    ytitle("Density") ///
    legend(label(1 "Control") label(2 "Treatment") rows(1)) ///
    graphregion(color(white))
```
