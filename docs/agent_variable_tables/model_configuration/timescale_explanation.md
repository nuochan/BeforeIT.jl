# How `timescale` Works in BeforeIT

## 1. The Core Problem It Solves

BeforeIT's calibration merges two data sources that operate at **different temporal frequencies**:

- **Input-Output (I-O) tables** (from FIGARO): **annual** data, measured in millions of euros per year
- **National Accounts / macroeconomic time series**: **quarterly** data

The model runs at **quarterly** frequency. So every flow variable derived from the annual I-O table must be converted to a quarterly equivalent. That's what `timescale` does.

## 2. The Formula

From `src/utils/calibration.jl` (lines 112–120):

```
timescale = nominal_gdp_quarterly / (
    sum(compensation_employees + operating_surplus + capital_consumption + taxes_production + taxes_products)
    + taxes_products_household
    + taxes_products_capitalformation_dwellings
    + taxes_products_government
    + taxes_products_export
)
```

Where:
- **Numerator**: Quarterly nominal GDP from National Accounts (a single scalar for the calibration quarter)
- **Denominator**: The sum of all annual I-O table components that constitute GDP from the income/production side — compensation of employees, operating surplus, capital consumption, taxes on production, and taxes on products (across all sectors plus final demand categories)

For Italy 2010Q1, this equals **≈ 0.156**, which makes sense: one quarter is roughly 1/4 of a year, but the ratio isn't exactly 0.25 because of **statistical discrepancies** between the I-O table and the National Accounts.

## 3. What It Converts (Every Use in the Code)

`timescale` appears in **~30 places** in the calibration. Here's a categorized breakdown:

### A. Sector-level productivity and cost parameters (per-employee, per-unit rates)

| Line | Variable | Formula | What it represents |
|------|----------|---------|-------------------|
| 175 | `alpha_s` | `timescale * output / employees` | **Labour productivity** per worker per quarter |
| 177 | `kappa_s` | `timescale * output / fixed_assets / omega` | **Capital productivity** per quarter |
| 178 | `delta_s` | `timescale * capital_consumption / fixed_assets / omega` | **Depreciation rate** per quarter |
| 180 | `w_s` | `timescale * wages / employees` | **Wage** per worker per quarter |

All four take an annual I-O flow (output, wages, capital consumption) and divide by a stock or count to get a **quarterly per-unit rate**. Without `timescale`, these would be annual rates — the model would pay workers their full annual salary every quarter.

### B. Financial rates (interest rates, markups)

| Line | Variable | Formula | What it represents |
|------|----------|---------|-------------------|
| 205 | `mu` | `timescale * firm_interest / firm_debt_quarterly - r_bar` | Bank **interest rate markup** per quarter |
| 242 | `r_G` | `timescale * interest_government_debt / government_debt_quarterly` | Government **bond yield** per quarter |

Here `firm_interest` and `interest_government_debt` are annual flows from the I-O table, while `firm_debt_quarterly` and `government_debt_quarterly` are already quarterly stocks. So `timescale` converts the annual interest payment to quarterly, then dividing by the stock gives a quarterly rate.

### C. Tax rate on firms (`tau_FIRM`)

Lines 210–220: Both the numerator (`corporate_tax`) and the denominator (`operating_surplus`, `firm_interest`) are annual I-O flows. `timescale` multiplies **both** numerator and denominator, so it **cancels out** — but the code keeps it explicit for clarity and consistency.

### D. Dividend share (`theta_DIV`)

Lines 230–240: Same pattern as `tau_FIRM` — `timescale` appears in both numerator and denominator of a ratio of annual flows, so it **cancels out** algebraically but is kept for structural consistency.

### E. Social transfers (per-person, per-quarter)

| Line | Variable | Formula |
|------|----------|---------|
| 355 | `w_UB` | `timescale * unemployment_benefits / unemployed` |
| 356 | `sb_inact` | `timescale * pension_benefits / inactive` |
| 358 | `sb_other` | `timescale * (social_benefits + ...) / (total population)` |

Annual transfer flows → quarterly per-person amounts.

### F. Macroeconomic time series (initial conditions & forecasts)

| Line | Variable | Formula pattern |
|------|----------|----------------|
| 369 | `Y` | `timescale * sum(output) * real_gdp_series / real_gdp_at_T` |
| 263 | `G_est` | `timescale * sum(gov_consumption) * real_gov_series / real_gov_at_T` |
| 269 | `E_est` | `timescale * sum(exports) * real_export_series / real_export_at_T` |
| 274 | `I_est` | `timescale * sum(imports) * real_import_series / real_import_at_T` |

These construct **quarterly-frequency time series** by:
1. Taking the annual I-O total (e.g., `sum(output)`)
2. Multiplying by `timescale` to get the quarterly level at the calibration date
3. Then scaling by the ratio of the real quarterly series to its value at the calibration date — this gives the historical trajectory at the correct quarterly scale

The same pattern is used for government spending forecasts (`C_G`), export forecasts (`C_E`), import forecasts (`Y_I`), and the EA GDP series (`Y_EA_series`).

## 4. Where `timescale` Does and Doesn't Apply

**Multiplied by `timescale`** (annual I-O flows → quarterly):
- Output, wages, operating surplus, capital consumption, interest payments, corporate tax, social benefits, government consumption, exports, imports

**NOT multiplied by `timescale`** (already quarterly or are ratios/shares):
- All **stock** variables: `D_I` (firm cash), `L_I` (firm debt), `D_H` (household deposits), `K_H` (housing), `L_G` (government debt), `E_k` (bank equity) — these are point-in-time balance sheet values, already quarterly
- All **share/ratio** parameters: `beta_s` (I-O coefficients), `tau_Y_s`, `tau_K_s` (tax rates as ratios of I-O flows), `b_CF_g`, `b_HH_g`, `a_sg`, `c_G_g`, `c_E_g`, `c_I_g` (expenditure shares) — ratios of annual-to-annual, so the frequency cancels
- **Tax rates** on households: `tau_INC`, `tau_VAT`, `tau_SIF`, `tau_SIW` — these are computed as ratios of I-O flows
- `tau_FIRM` and `theta_DIV` — `timescale` appears but cancels in the ratio
- `r_bar`, `rho`, `xi_pi`, `xi_gamma`, `pi_star` — estimated from quarterly time series directly

## 5. Why Not Just Divide by 4?

If the I-O table perfectly matched National Accounts, `timescale ≈ 0.25`. But it doesn't because:

1. **Statistical discrepancies**: The I-O table is compiled independently from the quarterly National Accounts, and they don't reconcile perfectly
2. **Balancing adjustments**: I-O tables are balanced to ensure rows = columns, introducing rounding differences
3. **Seasonal patterns**: GDP isn't uniformly distributed across quarters; the calibration quarter may be above or below the annual average
4. **Definitional differences**: Some components may use slightly different classifications between the I-O framework and the National Accounts

The actual value of ≈ 0.156 for Italy 2010Q1 (vs. the "naive" 0.25) shows a ~37% gap, meaning the I-O table total substantially exceeds 4× the quarterly GDP — likely because the I-O table records gross flows including double-counting of intermediate transactions that net out in the GDP identity.

## 6. Summary

`timescale` is a **single scalar** computed once during calibration. It serves as the universal bridge between the annual I-O table world and the quarterly simulation world. Every flow variable sourced from the I-O table is multiplied by it before being used as a model parameter or initial condition. Stock variables and pure ratios don't need it. It absorbs all statistical discrepancies between the two data sources into one consistent scaling factor, ensuring that when the model runs its first quarter, the simulated GDP matches the observed quarterly GDP.
