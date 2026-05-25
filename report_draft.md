# Sustainable Asset Allocation and Management
## Group AD — AMER + EUR | Scope 1 and Scope 2 Emissions
### SAAM Project 2026

---

## Abstract

This report examines the financial and carbon performance of five equity portfolios constructed from a universe of approximately 800 to 1,150 North American and European listed firms over the out-of-sample period January 2014 to December 2025. The two baseline portfolios are a value-weighted benchmark (VW) and a minimum-variance portfolio (MVP). Three carbon-constrained variants complete the study: an MVP constrained to halve its carbon footprint (MVP50), a VW portfolio minimising tracking error subject to a 50% carbon footprint reduction (VW50), and a decarbonisation-path portfolio anchored to the 2013 VW baseline with a 10% annual reduction target (VWNZ).

Over the full 144-month period, the VW benchmark delivered an annualised return of 11.45% with a Sharpe ratio of 0.680. The MVP reduced annualised volatility from 14.22% to 11.98%, but at a significant return cost: its annualised return was 6.16% with a Sharpe of 0.365. On carbon, the MVP did not naturally produce lower emissions. Its average carbon footprint of 149.1 tCO2 per million dollars invested is approximately 28% above the VW benchmark's 116.2, showing that variance minimisation and carbon reduction are not equivalent objectives.

The most practically useful strategy is the VW50, which halved the average carbon footprint from 116.2 to 57.7 tCO2/m$ at a cost of around 72 basis points of annual return. The VWNZ achieved similar financial performance to VW50, but delivered a higher average carbon footprint because its constraint was anchored to a 2013 baseline that became less restrictive as the market itself decarbonised over the sample period.

---

## 1. Introduction

The integration of climate risk into institutional portfolio management has become increasingly important over the past decade. The Paris Agreement, the Task Force on Climate-related Financial Disclosures (TCFD), and the European Sustainable Finance Disclosure Regulation (SFDR) have placed explicit pressure on asset managers to measure and reduce the carbon exposure of their portfolios. For practitioners and academics alike, a central question is how large the financial cost of this constraint actually is.

From a theoretical standpoint, any restriction added to the investment set can only tighten or leave unchanged the efficient frontier. In theory, carbon constraints carry a financial cost. The empirical question is how significant that cost is in practice, and whether the answer depends on the type of constraint chosen and the universe to which it is applied.

This project addresses that question using a real-data exercise for a universe of North American and European equities over 2014 to 2025. We construct five portfolios using monthly return and market capitalisation data from Refinitiv Datastream, combined with annual Scope 1 and Scope 2 carbon emissions and revenue data. The exercise has three main objectives. First, to characterise the financial performance of the VW benchmark and the MVP over the out-of-sample period. Second, to examine whether minimum-variance investing incidentally reduces carbon exposure. Third, to quantify the financial cost of explicit carbon constraints, both a static 50% reduction and a dynamic decarbonisation path.

The remainder of the report is structured as follows. Section 2 describes the data and investment universe. Section 3 presents the data cleaning steps and portfolio construction methodology. Section 4 covers carbon metrics and constrained optimisation. Section 5 reports and discusses the results. Section 6 discusses broader implications. Section 7 lists limitations. Section 8 concludes.

---

## 2. Data and Investment Universe

### 2.1 Data Sources

All data come from Refinitiv Datastream, 2025 vintage. The dataset covers 2,545 firms globally and includes both financial and carbon data.

**Table 1 — Data files**

| File | Content | Frequency |
|---|---|---|
| `DS_RI_T_USD_M_2025.xlsx` | Total return index (USD) | Monthly |
| `DS_MV_T_USD_M_2025.xlsx` | Market capitalisation (USD million) | Monthly |
| `DS_MV_T_USD_Y_2025.xlsx` | Market capitalisation (USD million) | Annual |
| `DS_CO2_SCOPE_1_Y_2025.xlsx` | Scope 1 CO2 emissions (tCO2) | Annual |
| `DS_CO2_SCOPE_2_Y_2025.xlsx` | Scope 2 CO2 emissions (tCO2) | Annual |
| `DS_REV_Y_2025.xlsx` | Revenue (USD thousand) | Annual |
| `Risk_Free_Rate_2025.xlsx` | Fama-French risk-free rate | Monthly |
| `Static_2025.xlsx` | ISIN, firm name, country, region | Static |

The monthly return index spans January 2000 to December 2025. Carbon data reports Scope 1 and Scope 2 greenhouse gas emissions in tonnes of CO2-equivalent per year. Revenue is stored in thousands of US dollars in the Datastream file and is divided by 1,000 before use to express it in millions, which is required for the carbon intensity computations. We do not use Scope 3 emissions, as coverage is incomplete and inconsistent across firms and years, particularly before 2018.

### 2.2 Investment Universe

The full Datastream file covers 2,545 firms globally. This project is assigned to Group AD, which covers the AMER and EUR regions. After applying the region filter using the `Static_2025.xlsx` country codes, the AMER+EUR universe contains approximately 1,300 firms. North America includes the United States and Canada. Europe includes France, Germany, the United Kingdom, the Netherlands, Switzerland, Italy, Spain, Sweden, and other European markets present in the database. The United States dominates the universe in market capitalisation terms, typically accounting for around 60 to 65 percent of the value-weighted benchmark weight over the sample period.

### 2.3 Eligible Investment Set

Each December from 2013 to 2024, we define an eligible investment set using five filters applied jointly.

**Table 2 — Eligibility filters**

| Filter | Condition |
|---|---|
| F1 | At least 36 non-missing monthly returns in the past 120 months |
| F2 | Non-missing total return index at the December formation date |
| F3 | Fraction of zero monthly returns below 50% in the 120-month window |
| F4 | At least one valid Scope 1 + Scope 2 observation up to the formation date |
| F5 | Not flagged as delisted |

F1 ensures there is sufficient return history to estimate the covariance matrix reliably. F3 screens out illiquid stocks whose prices rarely change, as a high fraction of zero returns inflates estimated volatility artificially. F4 ensures consistency with Part II: every firm in the universe has at least some carbon disclosure history, so the universe used for the financial analysis is the same as the one used for the carbon analysis. F5 removes firms that no longer trade.

The eligible universe grows from 813 firms in 2013 to over 1,150 by the early 2020s. The growth is mainly driven by F4: as sustainability disclosure requirements spread, more firms began reporting emissions data, progressively expanding the eligible set. The complete year-by-year count is in Appendix A.

---

## 3. Data Cleaning and Portfolio Construction Methodology

### 3.1 Data Cleaning

**Return computation.** Monthly simple returns are computed as:

$$R_{i,t} = \frac{\text{RI}_{i,t}}{\text{RI}_{i,t-1}} - 1$$

where $\text{RI}_{i,t}$ is the total return index of firm $i$ in month $t$. Any RI value below 0.5 is treated as missing before this computation, to avoid distorted returns when prices fall very close to zero. The `fill_method=None` option is used so that missing RI values are not silently forward-filled before differencing — doing so would generate spurious zero returns for firms with gaps in their price history.

**Delisting adjustment.** Datastream signals the delisting of a firm by appending the delisting date to the firm name field in the static file. We identify delisted firms using this flag and apply two corrections to the return index. The delisting month is assigned a return of -100%, representing a total loss for an investor who held until the end. All subsequent months are set to missing, as the firm no longer trades. Without this correction, Datastream's practice of carrying forward the last available price after delisting generates a misleading return profile: a spurious zero return in the month of disappearance, followed eventually by a large positive or negative return when the stale price is removed. This correction is especially important for portfolios that might hold small or distressed firms.

**Carbon data forward-fill.** Firms do not always report carbon emissions every year. Annual carbon and revenue observations are forward-filled along the year axis: if a firm reported in year $Y-k$ but not in year $Y$, the most recent observed value is carried forward. This reflects the assumption that emissions do not change dramatically year-to-year and that the last available report remains informative. We never fill backwards — if a firm has no carbon observation before year $Y$, its carbon record remains missing for all prior years, and it fails filter F4 until the first valid observation appears.

### 3.2 Covariance Matrix Estimation

For each formation year $Y$, the sample covariance matrix is estimated from the 120-month return history ending in December $Y$ of all eligible firms. Before computing the covariance matrix, any remaining missing monthly return is replaced by the cross-sectional mean return for that month — the average of all firms that had a valid return in that period. This imputation preserves the full time dimension without discarding any months and avoids biasing the covariance estimates by treating missing observations as zero returns. No shrinkage or factor model is applied. The covariance matrix is estimated using the maximum-likelihood estimator, dividing by $T$ rather than $T-1$, consistent with the project specification.

### 3.3 Minimum-Variance Portfolio

At each December formation date, we solve a standard long-only minimum-variance quadratic programme:

$$\min_{\boldsymbol{\alpha}} \; \boldsymbol{\alpha}^\top \hat{\boldsymbol{\Sigma}} \boldsymbol{\alpha}$$

$$\text{subject to} \quad \boldsymbol{1}^\top \boldsymbol{\alpha} = 1, \quad \boldsymbol{\alpha} \geq \boldsymbol{0}$$

where $\boldsymbol{\alpha} \in \mathbb{R}^n$ is the portfolio weight vector and $\hat{\boldsymbol{\Sigma}} \in \mathbb{R}^{n \times n}$ is the estimated sample covariance matrix. The long-only constraint eliminates short positions, which reduces sensitivity to estimation error in the covariance matrix and is consistent with a typical institutional mandate.

The problem is implemented using the CVXPY modelling library with the CLARABEL solver (CVXPY: Diamond and Boyd, 2016; CLARABEL: Goulart and Chen, 2023). For universes of 800 to 1,150 assets, each annual solve completes in under one second. The resulting portfolios are sparse: across the 12 formation years, between 24 and 42 firms receive non-zero weight. This concentration is typical of long-only minimum-variance solutions applied to large universes without additional diversification constraints.

### 3.4 Value-Weighted Benchmark

The VW benchmark assigns each firm a weight proportional to its market capitalisation, rebalanced monthly:

$$w_{i,t}^{\text{VW}} = \frac{\text{MV}_{i,t-1}}{\sum_{j \in \mathcal{U}_t} \text{MV}_{j,t-1}}$$

where $\text{MV}_{i,t-1}$ is the market capitalisation of firm $i$ at the end of the prior month from the monthly Datastream file, and $\mathcal{U}_t$ is the eligible investment set at date $t$. Weights are recomputed every month using actual Datastream data. This distinguishes the VW benchmark from a buy-and-hold approximation: under buy-and-hold, weights drift mechanically with price returns, while monthly rebalancing ensures the benchmark always reflects current relative market capitalisations.

### 3.5 Ex-Post Returns

**VW benchmark.** Each month, weights are computed from the prior month's actual market capitalisation and applied immediately. No drift formula is needed since weights are reset monthly.

**MVP, MVP50, VW50, and VWNZ.** These four portfolios use annual formation weights with buy-and-hold drift within the year. Weights are set at end-December $Y$ and drift passively over the following 12 months as stock prices move:

$$w_{i,t+1} = \frac{w_{i,t}(1 + R_{i,t+1})}{1 + R_{p,t+1}}, \quad \text{where} \quad R_{p,t+1} = \sum_j w_{j,t}(1 + R_{j,t+1}) - 1$$

If a firm's return is missing in a given month, its return is treated as zero in the weight-drift formula only, so the portfolio return is still computed from the firms with valid data.

**Timing convention.** Weights set at end-December $Y$ apply to returns from January $Y+1$ through December $Y+1$. The full out-of-sample period covers January 2014 to December 2025, for a total of 144 monthly return observations, spanning 12 formation years from end-2013 to end-2024.

### 3.6 Performance Metrics

Over the full 144-month period, we report the following statistics:

- **Annualised arithmetic return:** $\bar{R}_{\text{ann}} = 12 \times \bar{R}_m$, where $\bar{R}_m$ is the mean monthly return.
- **Annualised compounded return:** $R_{\text{cum}} = \left(\prod_{t=1}^{144}(1+R_t)\right)^{12/144} - 1$.
- **Annualised volatility:** $\sigma_{\text{ann}} = \sqrt{12} \times \sigma_m$, where $\sigma_m$ is the standard deviation of monthly returns.
- **Sharpe ratio:** $(12 \times \bar{R}_m - \bar{R}_f^{\text{ann}}) \,/\, \sigma_{\text{ann}}$, where $\bar{R}_f^{\text{ann}}$ is the annualised mean risk-free rate from the Fama-French file, converted from percent to decimal and aligned monthly to the portfolio return series.
- **Minimum and maximum monthly return:** direct minimum and maximum of the monthly series.

---

## 4. Carbon Metrics and Constrained Optimisation

### 4.1 Carbon Footprint

The carbon footprint (CF) of a portfolio at formation year $Y$ is defined as:

$$\text{CF}_Y(\boldsymbol{\alpha}) = \sum_{i=1}^{n} \alpha_i \cdot \frac{E_i^Y}{\text{Cap}_i^Y}$$

where $E_i^Y$ is firm $i$'s total Scope 1 plus Scope 2 emissions in tonnes of CO2-equivalent for year $Y$, and $\text{Cap}_i^Y$ is firm $i$'s annual market capitalisation in millions of USD. The result is expressed in tCO2 per million dollars invested. This metric captures the emissions that the investor indirectly finances, scaled by equity value — the higher the portfolio weight in a firm with high emissions relative to its market cap, the higher the portfolio's carbon footprint.

### 4.2 Weighted-Average Carbon Intensity

The weighted-average carbon intensity (WACI) is defined as:

$$\text{WACI}_Y(\boldsymbol{\alpha}) = \sum_{i=1}^{n} \alpha_i \cdot \frac{E_i^Y}{\text{Rev}_i^Y}$$

where $\text{Rev}_i^Y$ is firm $i$'s revenue in millions of USD. Unlike CF, WACI measures how carbon-efficient a firm is per dollar of economic output, independently of its market valuation. WACI is therefore less sensitive to market price fluctuations and more closely reflects operational carbon efficiency.

Both metrics use the forward-filled annual emissions and revenue data described in Section 3. For firms that appear in the portfolio but have a missing carbon observation in the optimiser constraint, carbon intensity is set to zero. This is a simplifying assumption: it means the constraint does not treat firms with missing data as high emitters, which likely understates the true constrained portfolio footprint. This limitation is discussed in Section 7.

### 4.3 Carbon-Constrained MVP (MVP50)

The MVP50 adds a single carbon constraint to the standard minimum-variance problem:

$$\min_{\boldsymbol{\alpha}} \; \boldsymbol{\alpha}^\top \hat{\boldsymbol{\Sigma}} \boldsymbol{\alpha}$$

$$\text{subject to} \quad \boldsymbol{1}^\top \boldsymbol{\alpha} = 1, \quad \boldsymbol{\alpha} \geq \boldsymbol{0}, \quad \text{CF}_Y(\boldsymbol{\alpha}) \leq 0.5 \times \text{CF}_Y(\boldsymbol{\alpha}^{\text{MVP}})$$

The carbon budget is set to half the unconstrained MVP's footprint at the same formation date. Because $\text{CF}_Y(\boldsymbol{\alpha})$ is linear in $\boldsymbol{\alpha}$, this constraint preserves the quadratic structure of the problem and does not increase the computation time significantly.

### 4.4 VW Portfolio with 50% Carbon Reduction (VW50)

The VW50 minimises tracking error relative to the VW benchmark while halving its carbon footprint:

$$\min_{\boldsymbol{\alpha}} \; (\boldsymbol{\alpha} - \boldsymbol{w}^{\text{VW}})^\top \hat{\boldsymbol{\Sigma}} (\boldsymbol{\alpha} - \boldsymbol{w}^{\text{VW}})$$

$$\text{subject to} \quad \boldsymbol{1}^\top \boldsymbol{\alpha} = 1, \quad \boldsymbol{\alpha} \geq \boldsymbol{0}, \quad \text{CF}_Y(\boldsymbol{\alpha}) \leq 0.5 \times \text{CF}_Y(\boldsymbol{w}^{\text{VW}})$$

where $\boldsymbol{w}^{\text{VW}}$ is the annual VW weight vector computed from year-end market capitalisations. Minimising tracking error rather than total variance keeps the constrained portfolio close to the benchmark in terms of sector and factor exposures, limiting unintended active bets outside the carbon objective.

### 4.5 Net-Zero Decarbonisation Path (VWNZ)

The VWNZ uses a tightening carbon budget anchored to the 2013 VW baseline, declining at 10% per year on a compound basis:

$$\text{CF}_Y(\boldsymbol{\alpha}) \leq (1 - 0.10)^{Y - 2013 + 1} \times \text{CF}^{\text{VW}}_{2013}$$

where $\text{CF}^{\text{VW}}_{2013} = 183.84$ tCO2/m$ is the 2013 VW carbon footprint. In formation year 2013, the budget is $0.9 \times 183.84 = 165.5$ tCO2/m$. By formation year 2024, the budget is $(0.9)^{12} \times 183.84 \approx 51.9$ tCO2/m$, a reduction of approximately 72% from the 2013 baseline. The label "net-zero" is shorthand for a Paris-aligned decarbonisation glide path in the spirit of Andersson, Bolton, and Samama (2016). It does not bring the portfolio to zero emissions over the sample period. The objective is the same as VW50: minimise tracking error to the VW benchmark, subject to the carbon budget above.

---

## 5. Results

### 5.1 Part I: VW Benchmark versus Minimum-Variance Portfolio

Table 3 reports performance statistics for the VW benchmark and the MVP over the full 144-month out-of-sample period.

**Table 3 — Performance: VW and MVP (January 2014 to December 2025)**

| Metric | VW | MVP |
|---|---|---|
| Ann. arithmetic return | **11.45%** | 6.16% |
| Ann. compounded return | 10.96% | 5.58% |
| Ann. volatility | 14.22% | **11.98%** |
| Sharpe ratio | **0.680** | 0.365 |
| Min. monthly return | -13.20% | -12.73% |
| Max. monthly return | 12.94% | 11.50% |

The VW benchmark strongly outperforms the MVP over this period. Its annualised arithmetic return of 11.45% is 529 basis points above the MVP's 6.16%, and its Sharpe ratio of 0.680 is nearly double the MVP's 0.365. The period 2014 to 2025 was characterised by a sustained rally in large-cap US technology stocks — Apple, Microsoft, Nvidia, Alphabet, and Amazon among others — which carry substantial weight in a cap-weighted benchmark and delivered exceptional returns. The MVP, by construction, avoids volatile and highly correlated assets, which includes most of the technology sector, and this structural underweight was costly over this particular period.

The MVP does achieve its stated objective of reducing portfolio variance. Its annualised volatility of 11.98% is 224 basis points below the VW benchmark's 14.22%. Its worst monthly return of -12.73% is marginally better than the VW's -13.20%, consistent with the lower volatility. However, the return sacrifice of 529 basis points per year is large relative to the volatility improvement of 224 basis points, producing a Sharpe ratio well below the benchmark. This is a consequence of the sample period rather than a general result: in a market environment where the highest-returning assets are also among the most volatile, a minimum-variance approach will structurally underperform.

Figure 1 shows the cumulative performance of the two portfolios over the full sample period.

*(Figure 1: `outputs/figures/cumulative_returns_part1.png`)*

### 5.2 Part II: Carbon Footprint and WACI of Unconstrained Portfolios

Table 4 reports the average annual carbon footprint and WACI for all five portfolios.

**Table 4 — Average annual carbon metrics (2014 to 2025)**

| Portfolio | Avg. CF (tCO2/m$) | Avg. WACI (tCO2/m$ revenue) |
|---|---|---|
| VW | 116.2 | 168.0 |
| MVP | 149.1 | 290.0 |
| MVP50 | 74.5 | 137.5 |
| VW50 | 57.7 | 85.5 |
| VWNZ | 96.8 | 129.2 |

A notable finding from the unconstrained portfolios is that the MVP has a higher carbon footprint than the VW benchmark, not lower. On average, the MVP's carbon footprint is 149.1 tCO2/m$ versus 116.2 for the VW benchmark, approximately 28% higher. Its WACI is 290.0 tCO2/m$ revenue, compared to 168.0 for the VW, a relative gap of around 73%.

This result illustrates that minimum-variance investing does not automatically produce a low-carbon portfolio. The MVP optimises purely on estimated return variance and tends to concentrate on stocks with stable, low-volatility earnings profiles. In this AMER+EUR universe over this period, some of those firms — particularly regulated utilities and certain industrial companies — also happen to have above-average carbon intensity relative to their market cap or revenue. The result is that the MVP's carbon exposure ends up above the market benchmark. An investor who expects minimum variance to serve as an incidental carbon reduction strategy would be misled.

Figure 2 shows the evolution of carbon footprint and WACI over time for all five portfolios.

*(Figure 2: `outputs/figures/carbon_metrics_evolution.png`)*

### 5.3 Carbon-Constrained MVP (MVP50)

Adding a 50% carbon footprint constraint to the MVP produces the following results. The average carbon footprint falls from 149.1 to 74.5 tCO2/m$, exactly as required by the constraint. The average WACI falls from 290.0 to 137.5. Financially, the annual return drops from 6.16% to 5.92%, the Sharpe ratio falls from 0.365 to 0.348, and volatility decreases marginally from 11.98% to 11.86%.

The carbon reduction is effective: the constraint halves the footprint and also substantially reduces the WACI. The financial cost is modest: a Sharpe ratio reduction of 0.017 and an annual return reduction of 24 basis points. This is consistent with theory: any binding constraint on the investment set removes some flexibility and weakens the unconstrained optimum. In this case the cost is limited, but the direction is as expected — the constrained portfolio delivers slightly lower risk-adjusted performance than the unconstrained one.

*(Figure 3: `outputs/figures/cum_mvp_vs_mvp50.png`)*

### 5.4 VW Portfolio with 50% Carbon Reduction (VW50)

The VW50 portfolio is the central practical result of this project. Table 5 summarises the trade-off relative to the unconstrained VW benchmark.

**Table 5 — VW versus VW50 trade-off**

| Metric | VW | VW50 | Change |
|---|---|---|---|
| Ann. arithmetic return | 11.45% | 10.73% | -72 bps |
| Ann. volatility | 14.22% | 14.40% | +18 bps |
| Sharpe ratio | 0.680 | 0.621 | -0.059 |
| Avg. CF (tCO2/m$) | 116.2 | 57.7 | -50.3% |
| Avg. WACI (tCO2/m$ rev.) | 168.0 | 85.5 | -49.1% |

The annual return falls from 11.45% to 10.73%, a difference of approximately 72 basis points. This gap should be read carefully: the VW benchmark rebalances monthly, while the VW50 is formed annually and held with buy-and-hold drift within each year. Part of the performance difference may therefore reflect the change in rebalancing frequency rather than the carbon constraint alone; without an unconstrained annual-formation VW baseline, the two effects cannot be separated. The Sharpe ratio falls from 0.680 to 0.621. Volatility increases marginally by 18 basis points. In exchange, the average carbon footprint is halved from 116.2 to 57.7 tCO2/m$, and the WACI is nearly halved from 168.0 to 85.5.

The tracking error of the VW50 relative to the VW benchmark is near-zero across all 12 formation years. This is a genuine finding rather than a numerical artefact: with 800 to 1,150 stocks in the eligible universe, each individually high-carbon firm accounts for less than 1.5% of total market capitalisation. Removing these names and redistributing their weight across hundreds of lower-carbon stocks changes the portfolio composition by very little, so the tracking-error objective is nearly fully satisfied by this marginal reweighting.

One tail-risk consideration is worth noting: the VW50's worst monthly return is -15.05%, materially worse than the VW benchmark's -13.20%. An annually formed portfolio cannot rebalance ahead of a market stress event, which can cause the portfolio to diverge further from its target weights during sharp drawdowns.

For investors subject to EU SFDR obligations, TCFD reporting requirements, or internal net-zero commitments, the measured return difference of around 72 basis points per year — an upper bound on the carbon cost — for a 50% reduction in financed emissions is a manageable cost. The VW50 stays structurally close to the VW benchmark — it minimises tracking error — so it does not take large active bets outside the carbon objective. It is the most defensible climate-aware strategy in this exercise.

*(Figure 4: `outputs/figures/cum_vw_variants.png`)*

### 5.5 VW Net-Zero Path (VWNZ)

The VWNZ portfolio uses a tightening carbon budget anchored to the 2013 VW baseline. Its financial performance is close to VW50: an annualised return of 10.78% and a Sharpe ratio of 0.625, both slightly above VW50. However, its average carbon footprint is 96.8 tCO2/m$, notably higher than VW50's 57.7.

This gap is explained by the design of the constraint. The VWNZ uses a fixed 2013 baseline of 183.84 tCO2/m$ and applies a 10% annual compound reduction. In the early years of the sample, this constraint is relatively relaxed. For example, the 2014 budget is $0.9 \times 183.84 = 165.5$ tCO2/m$, while the VW benchmark itself was already well below that level by then. By contrast, the VW50 always targets 50% of the current VW benchmark footprint, which declined from approximately 184 tCO2/m$ in 2013 to around 60 by 2025 as the market itself decarbonised. In the later years of the sample, the VW50 constraint becomes much tighter than the VWNZ constraint. The VWNZ is therefore less constrained on average, which allows a broader and slightly more diversified portfolio — explaining the marginally better financial performance — but at the cost of a higher average carbon footprint.

The full performance and carbon summary for all five portfolios is in Table 6.

**Table 6 — Full results: all five portfolios (January 2014 to December 2025)**

| Metric | VW | MVP | MVP50 | VW50 | VWNZ |
|---|---|---|---|---|---|
| Ann. arithmetic return | 11.45% | 6.16% | 5.92% | 10.73% | 10.78% |
| Ann. compounded return | 10.96% | 5.58% | 5.34% | 10.14% | 10.19% |
| Ann. volatility | 14.22% | 11.98% | 11.86% | 14.40% | 14.40% |
| Sharpe ratio | 0.680 | 0.365 | 0.348 | 0.621 | 0.625 |
| Min. monthly return | -13.20% | -12.73% | -12.23% | -15.05% | -15.07% |
| Max. monthly return | 12.94% | 11.50% | 11.32% | 13.41% | 13.37% |
| Avg. CF (tCO2/m$) | 116.2 | 149.1 | 74.5 | 57.7 | 96.8 |
| Avg. WACI (tCO2/m$ rev.) | 168.0 | 290.0 | 137.5 | 85.5 | 129.2 |

*(Figure 5: `outputs/figures/cum_all_portfolios.png`)*

---

## 6. Discussion

The main finding of this project is that a 50% reduction in carbon footprint relative to the VW benchmark is achievable at a measured annual return difference of around 72 basis points. This figure should be qualified: the VW benchmark rebalances monthly while the VW50 forms annually, and the rebalancing frequency change contributes to the performance gap alongside the carbon constraint. The true cost of the carbon constraint alone is therefore likely lower than 72 basis points. Subject to this caveat, for most institutional investors today this is a limited and defensible price for a clearly measured carbon outcome. This result is specific to the AMER+EUR universe and the 2014 to 2025 period, but it is consistent with the broader literature suggesting that carbon constraints applied to large and well-diversified universes tend to have modest financial consequences (Andersson et al., 2016).

A secondary finding concerns the relationship between minimum-variance investing and carbon exposure. In this universe, the MVP's carbon footprint and WACI are both higher than the VW benchmark's. This is not a dramatic difference — the MVP's CF is 28% above the VW — but it is directionally clear. Minimum-variance investing, as implemented here, tends to concentrate weight on a small number of low-volatility stocks that are not systematically low-carbon. Investors should not assume that variance minimisation is a substitute for an explicit carbon constraint.

The comparison between VW50 and VWNZ illustrates an important design consideration for net-zero mandates. Both portfolios use the same objective — minimise tracking error to the VW benchmark — and delivered similar financial performance over the sample. But their carbon outcomes differ meaningfully. The VW50, which always targets 50% of the current benchmark footprint, produced an average carbon footprint of 57.7 tCO2/m$ over the full period. The VWNZ, anchored to a 2013 baseline, produced 96.8 on average. The reason is that the market itself decarbonised substantially over the period, which made the historically anchored VWNZ constraint progressively less tight relative to current benchmark levels. Investors designing long-run decarbonisation mandates should be explicit about whether their reference is absolute, relative to a fixed historical baseline, or relative to the current benchmark, as this choice has significant consequences over time.

---

## 7. Limitations

**Covariance estimation.** We use the plain sample covariance matrix over a 120-month window without shrinkage or regularisation. For universes of 800 to 1,150 assets, this produces a noisy matrix with many near-zero eigenvalues, and the unconstrained MVP concentrates all weight on a small number of positions — typically 24 to 42 firms out of 800 to 1,150 eligible. A Ledoit-Wolf shrinkage estimator or a factor-based covariance model would likely produce a more diversified and stable MVP, but would also complicate the methodology and make results harder to compare to the project baseline. We therefore accept this limitation in exchange for methodological simplicity.

**Carbon data quality.** All emissions data are self-reported by firms to the Datastream ESG database. Reporting standards differ across countries and over time, and early-sample data before 2015 is sparser and less independently audited. Some year-level extreme values visible in the carbon time series are likely data artefacts rather than genuine changes in firm emissions.

**Missing carbon data in the optimiser.** In the carbon constraint, firms without a valid carbon observation are assigned a carbon intensity of zero. This means the optimiser does not penalise them as if they were carbon-intensive, which likely understates the constrained portfolio's true footprint. A more conservative approach would exclude such firms from the constrained universe, but this would reduce the eligible set and potentially introduce a different form of bias. We treat this as a limitation rather than a feature.

**Scope 3 excluded.** We use only Scope 1 and Scope 2 emissions. For oil and gas companies, the largest share of their climate exposure lies in Scope 3 — the CO2 released when customers burn the fuel they produce. For banks, the most relevant climate exposure is financed emissions in their lending book. Including Scope 3 would give a more complete picture of portfolio climate risk, but data coverage and quality are insufficient for reliable use in an optimisation at this stage.

**No transaction costs.** The analysis ignores transaction costs. Annual reoptimisation generates turnover, particularly for the VWNZ in later years as the carbon budget tightens. In practice, round-trip transaction costs would reduce but not reverse the main conclusions, particularly for the VW50, where the carbon constraint is static from year to year.

**Rebalancing frequency confound.** The VW benchmark is rebalanced monthly, while the VW50 and VWNZ portfolios are formed annually and held with buy-and-hold drift. The 72 basis-point annual return gap between VW and VW50 therefore combines the effect of the carbon constraint with the effect of less frequent rebalancing. Without an unconstrained annual-formation VW baseline, the contribution of each factor cannot be isolated. The true cost of the carbon constraint alone is likely smaller than 72 basis points.

**Single sample period.** The results cover one 12-year window, which was unusual in several respects: a sustained US technology rally and a general trend of market-level decarbonisation. The VW outperformed the MVP strongly in this environment, and the VW benchmark's own declining carbon footprint made the VWNZ constraint progressively less binding. Results from a different period — particularly one with weaker technology returns or rising utility valuations — could look meaningfully different.

**No statistical tests.** Performance differences are not tested for statistical significance. With a single non-overlapping 12-year sample, formal tests would have limited power. All comparisons should be read as descriptive.

**Carbon constraints and real-world emissions.** Reducing the portfolio's measured carbon footprint does not directly reduce a company's physical emissions. The real-economy impact of portfolio-level reweighting is indirect and contested in the literature. Our carbon metrics measure financed exposure, not actual emissions abatement.

---

## 8. Conclusion

This project builds and evaluates five equity portfolios for a universe of North American and European firms over January 2014 to December 2025.

The first main finding is that the VW benchmark substantially outperforms the MVP in financial terms. The MVP achieves a meaningful volatility reduction — from 14.22% to 11.98% — and slightly better tail protection, but its annualised return is 529 basis points below the VW benchmark, and its Sharpe ratio is roughly half that of VW. This large performance gap reflects the dominance of large-cap US technology stocks over the sample period, a sector that the MVP structurally underweights.

The second finding is that minimum-variance investing does not reduce carbon exposure in this universe. The MVP's average carbon footprint of 149.1 tCO2/m$ is approximately 28% above the VW benchmark's 116.2, and its WACI of 290.0 is 73% above the benchmark's 168.0. These results show that variance minimisation and carbon reduction are distinct objectives. If carbon reduction is a goal, an explicit constraint is required.

The third and most practically useful finding concerns the VW50. A 50% carbon footprint constraint applied within a tracking-error-minimising framework reduced the average carbon footprint from 116.2 to 57.7 tCO2/m$ at an observed annual return difference of approximately 72 basis points — an upper bound on the carbon constraint cost, since the VW50's annual formation is not directly comparable to the VW benchmark's monthly rebalancing. The Sharpe ratio fell from 0.680 to 0.621. This trade-off is the central result: a large, clearly defined carbon reduction at a limited and defensible financial cost.

The fourth finding concerns net-zero mandate design. The VWNZ portfolio, which anchors its carbon budget to a 2013 baseline, delivered similar financial performance to VW50 but a meaningfully higher average carbon footprint. As the VW benchmark itself decarbonised over the sample period, the historically anchored constraint became progressively less restrictive than the current-benchmark 50% constraint. Investors implementing net-zero mandates need to be precise about their reference point: a fixed historical anchor becomes less ambitious over time in a decarbonising market.

---

## Use of Large Language Models

We used ChatGPT and Claude as support tools during this project, mainly to discuss coding errors, improve code readability, check the clarity of methodological explanations, and polish grammar and structure in the written report. The portfolio construction choices, data cleaning decisions, implementation, result generation, interpretation of the findings, and final conclusions were reviewed and validated by the group. We remain fully responsible for the correctness of the code, results, and submitted work.

---

## References

Andersson, M., Bolton, P., and Samama, F. (2016). Hedging climate risk. *Financial Analysts Journal*, 72(3), 13-32.

Diamond, S., and Boyd, S. (2016). CVXPY: A Python-embedded modeling language for convex optimization. *Journal of Machine Learning Research*, 17(83), 1-5.

Goulart, P., and Chen, Y. (2023). CLARABEL: An interior-point solver for conic programs with quadratic objectives. *arXiv preprint arXiv:2304.06271*.

Gourier, E., Jondeau, E., and Rockinger, M. (2024). *Sustainable Asset Allocation and Management: Course Notes*. HEC Lausanne.

GHG Protocol (2015). *Corporate Value Chain (Scope 3) Accounting and Reporting Standard*. World Resources Institute and World Business Council for Sustainable Development.

Markowitz, H. (1952). Portfolio selection. *Journal of Finance*, 7(1), 77-91.

Refinitiv Datastream (2025). *ESG and financial data, 2025 vintage*. London Stock Exchange Group.

TCFD (2017). *Recommendations of the Task Force on Climate-related Financial Disclosures*. Financial Stability Board.

---

## Appendix A — Eligible Investment Set by Year

| Year | Eligible firms |
|---|---|
| 2013 | 813 |
| 2014 | 842 |
| 2015 | 876 |
| 2016 | 900 |
| 2017 | 932 |
| 2018 | 989 |
| 2019 | 1,083 |
| 2020 | 1,131 |
| 2021 | 1,168 |
| 2022 | 1,169 |
| 2023 | 1,154 |
| 2024 | 1,140 |

### Appendix B — Net-Zero Annual Carbon Targets

Annual carbon budget targets for the VWNZ portfolio. Baseline: 183.84 tCO2/m$, compounded reduction of 10% per year.

| Formation year | Cumulative reduction | Budget (tCO2/m$) |
|---|---|---|
| 2013 | -10% | 165.5 |
| 2014 | -19% | 148.9 |
| 2015 | -27% | 134.0 |
| 2016 | -34% | 120.6 |
| 2017 | -41% | 108.6 |
| 2018 | -47% | 97.7 |
| 2019 | -52% | 87.9 |
| 2020 | -57% | 79.1 |
| 2021 | -61% | 71.2 |
| 2022 | -65% | 64.1 |
| 2023 | -69% | 57.7 |
| 2024 | -72% | 51.9 |

---

*Group AD | AMER + EUR | Scope 1 and Scope 2 | SAAM Project 2026*
*Data: Refinitiv Datastream, 2025 vintage. Out-of-sample period: January 2014 to December 2025.*
