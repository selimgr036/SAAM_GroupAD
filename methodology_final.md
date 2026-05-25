# SAAM Project — Methodology Notes
## Group AD | AMER + EUR | Scope 1 + Scope 2

---

## Overview

The project builds five portfolios over a 12-year out-of-sample period (January 2014 to December 2025). The first two are the baseline portfolios from Part I. The last three add carbon constraints in Part II.

| Portfolio | Description |
|---|---|
| **VW** | Value-weighted benchmark, monthly market-cap rebalanced |
| **MVP** | Minimum-variance portfolio, long-only, annual rebalancing |
| **MVP50** | MVP constrained to halve its carbon footprint |
| **VW50** | Minimises tracking error vs VW, constrained to cut CF by 50% |
| **VWNZ** | Minimises tracking error vs VW, 10% annual decarbonisation path from 2013 baseline |

All portfolios use the same eligible universe each year. The VW benchmark is rebalanced monthly using actual market-cap data. The four other portfolios (MVP, MVP50, VW50, VWNZ) use annual formation weights with buy-and-hold weight drift within each year.

---

## Data

### Files used

| File | What it contains |
|---|---|
| `Static_2025.xlsx` | 2,545 firms: ISIN, name, country, region |
| `DS_RI_T_USD_M_2025.xlsx` | Monthly total return index (USD), 2000-2025 |
| `DS_MV_T_USD_M_2025.xlsx` | Monthly market cap (million USD), 2000-2025 |
| `DS_MV_T_USD_Y_2025.xlsx` | Annual market cap (million USD), 2000-2024 |
| `DS_CO2_SCOPE_1_Y_2025.xlsx` | Annual Scope 1 CO2 emissions (tonnes), 2000-2024 |
| `DS_CO2_SCOPE_2_Y_2025.xlsx` | Annual Scope 2 CO2 emissions (tonnes), 2000-2024 |
| `DS_REV_Y_2025.xlsx` | Annual revenue (thousands USD), 2000-2024 |
| `Risk_Free_Rate_2025.xlsx` | Monthly Fama-French risk-free rate (percent) |

### File format

All Datastream files share the same layout: row 0 is the real header with firm names, ISINs, and time periods (Datastream date strings parseable by pandas). Loading with `header=0` reads this row as the column header.

---

## Step-by-step walkthrough

### 1. Filter to AMER + EUR

Keep only the approximately 1,300 firms whose ISIN appears in `Static_2025.xlsx` with `Region` in `{AMER, EUR}`.

---

### 2. Low RI filter

Before computing returns, any total return index value below 0.5 is set to NaN. This prevents extreme return values when a price is close to zero and avoids distorting the covariance matrix.

---

### 3. Monthly returns

Simple returns are computed as:

$$R_{i,t} = \frac{\text{RI}_{i,t}}{\text{RI}_{i,t-1}} - 1$$

The `fill_method=None` option is passed to `pct_change` so that missing RI values are not forward-filled before differencing.

---

### 4. Delist corrections

Datastream encodes delistings in the firm name. For identified delisted firms, we apply two corrections:
- At the delist month: set RI to 0 (return = -100%)
- After the delist month: set RI to NaN

---

### 5. Forward-filling annual carbon and revenue data

Carbon observations are forward-filled along the year axis after the first valid observation. Revenue is also divided by 1,000 to convert from thousands USD to millions USD.

---

### 6. Investment set construction

At end of December Y (from 2013 to 2024), a firm must pass five criteria:

| # | Criterion | Why |
|---|---|---|
| F1 | At least 36 non-NaN monthly returns in the past 120 months | Need sufficient history for covariance estimation |
| F2 | Non-NaN RI at December Y | Current price must exist |
| F3 | Fraction of zero returns below 50% in the 120-month window | Illiquid stocks inflate estimated volatility |
| F4 | Has at least one Scope 1+2 observation up to year Y | Consistency with Part II |
| F5 | Not delisted on or before December Y | Firm must still trade |

This gives between 812 and 1,169 eligible firms per year. The binding constraint in early years is F4, which relaxes as sustainability disclosure spreads.

---

### 7. Covariance matrix estimation

For each year Y, using the eligible firms and their returns over the past 120 months:

1. Fill any remaining NaN monthly returns with the cross-sectional mean for that month.
2. Compute the sample covariance matrix from the filled matrix.

No shrinkage is applied. The long-only constraint on the MVP partially mitigates the sensitivity to covariance estimation error.

---

### 8. Minimum-variance portfolio (MVP)

$$\min_\alpha \; \alpha^\top \Sigma_Y \alpha \quad \text{subject to} \quad \mathbf{1}^\top\alpha = 1, \;\; \alpha \geq 0$$

Solved with CVXPY and the CLARABEL solver. The result is sparse: 24 to 42 firms typically receive non-zero weights across the 12 formation years.

---

### 9. Ex-post returns (buy-and-hold)

Weights are set at end of December Y and held throughout year Y+1 without rebalancing. The weight of firm i drifts each month as prices change:

$$w_{i,t+1} = \frac{w_{i,t}(1 + R_{i,t+1})}{1 + R_{p,t+1}}$$

Weights from December Y apply to returns in January Y+1 through December Y+1.

---

### 10. Value-weighted benchmark (VW)

Each month, weights are computed from the prior month's actual Datastream market capitalisation:

$$w_{i,t}^{\text{VW}} = \frac{\text{MV}_{i,t-1}}{\sum_j \text{MV}_{j,t-1}}$$

The VW benchmark is rebalanced monthly. This is distinct from the buy-and-hold drift used for the other four portfolios.

---

### 11. Performance statistics

Over the full 144-month period:

| Statistic | Formula |
|---|---|
| Annualised avg. return | `mean(R) × 12` |
| Annualised compounded return | `(∏(1+R_t))^(12/144) − 1` |
| Annualised volatility | `std(R) × √12` |
| Sharpe ratio | `(mean(R − RF) × 12) / (std(R) × √12)` |
| Min / Max monthly | direct min/max |

---

## Part II — Carbon Metrics

### 12. Carbon footprint and WACI

**Carbon Footprint (CF)**:
$$\text{CF}^p_Y = \sum_i \alpha_{i,Y} \cdot \frac{E_{i,Y}}{\text{Cap}_{i,Y}}$$

Units: tCO2/m$ invested.

**WACI**:
$$\text{WACI}^p_Y = \sum_i \alpha_{i,Y} \cdot \frac{E_{i,Y}}{\text{Rev}_{i,Y}}$$

Units: tCO2/m$ revenue.

Both metrics use forward-filled annual data. For the CF constraint in the optimiser, firms with missing emissions or market cap data are treated with intensity zero. This is a simplification — it means firms without data are not penalised in the optimiser — and is acknowledged as a limitation.

---

### 13. MVP50 — MVP with 50% carbon footprint reduction

$$\min_\alpha \alpha^\top \Sigma_Y \alpha \quad \text{subject to} \quad \sum_i \alpha_i \cdot \frac{E_i}{\text{Cap}_i} \leq 0.5 \cdot \text{CF}(\alpha^{\text{MVP}})_Y, \quad \mathbf{1}^\top\alpha = 1, \;\; \alpha \geq 0$$

The constraint halves the unconstrained MVP's footprint. It imposes a modest financial cost: Sharpe falls from 0.365 to 0.348, and annual return from 6.16% to 5.92%. This is the expected direction: any binding constraint removes flexibility.

---

### 14. VW50 — Tracking error minimisation with 50% CF reduction

$$\min_\alpha (\alpha - \alpha^{\text{VW}}_Y)^\top \Sigma_Y (\alpha - \alpha^{\text{VW}}_Y) \quad \text{subject to} \quad \sum_i \alpha_i \cdot \frac{E_i}{\text{Cap}_i} \leq 0.5 \cdot \text{CF}(\alpha^{\text{VW}})_Y, \quad \mathbf{1}^\top\alpha = 1, \;\; \alpha \geq 0$$

The target is 50% of the VW carbon footprint at year Y. Annual return cost: approximately 72 basis points (11.45% → 10.73%). This is the main practical result of the project.

---

### 15. VWNZ — Net-zero glide path

$$\text{CF}(\alpha_Y) \leq (1 - 0.10)^{Y - 2013 + 1} \cdot \text{CF}^{\text{VW}}_{2013}$$

Baseline: CF_VW_2013 = 183.84 tCO2/m$. The budget declines from 165.5 in 2013 to 51.9 by 2024. The VWNZ delivers similar financial performance to VW50, but a higher average carbon footprint (96.8 vs 57.7), because the historically anchored constraint becomes less restrictive as the market itself decarbonises.

---

## Design choices

**Why buy-and-hold within the year?** Standard passive assumption for annual-rebalancing portfolios. Reduces turnover and is consistent with the project specification.

**Why annual rebalancing?** Carbon data is annual. Annual rebalancing aligns the portfolio with the frequency of the data used for the carbon constraint.

**Why CLARABEL?** Fast interior-point solver for quadratic programmes. Handles universes of 800-1,150 variables efficiently.

**Why forward-fill carbon data?** Carbon reports have a 12-18 month lag. Forward-filling gives a stable carbon intensity estimate without introducing look-ahead bias. We never fill backwards.

**Why the same eligible universe for all portfolios?** Consistency: performance differences should reflect the optimisation, not differences in stock selection.

---

## Limitations

1. **Estimation risk**: plain sample covariance without shrinkage. The MVP is over-concentrated (24-42 positions).

2. **Carbon data quality**: self-reported, inconsistent before 2015. Some extreme year-level values are likely artefacts.

3. **Missing carbon data**: firms without a carbon observation are assigned zero intensity in the optimiser. This is a simplification, not an assumption of cleanliness.

4. **Transaction costs**: ignored. Annual reoptimisation generates non-trivial turnover.

5. **Scope 3 excluded**: supply-chain and downstream emissions omitted due to data quality.

6. **Single period**: 2014-2025 was an unusual period for US technology stocks. Results are period-specific.
