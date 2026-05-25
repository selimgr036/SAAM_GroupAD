# SAAM Part I — What We Did and Why

**Group AD | North America + Europe | Scope 1+2**

---

## Files needed

| File | What it is |
|---|---|
| `Static_2025.xlsx` | List of all 2545 firms with their ISIN, name, country, and region |
| `DS_RI_T_USD_M_2025.xlsx` | Monthly price index (total return) for each firm, 1999–2025 |
| `DS_MV_T_USD_M_2025.xlsx` | Monthly market cap for each firm, in million USD |
| `DS_CO2_SCOPE_1_Y_2025.xlsx` | Annual Scope 1 CO2 emissions per firm, in tonnes |
| `DS_CO2_SCOPE_2_Y_2025.xlsx` | Annual Scope 2 CO2 emissions per firm, in tonnes |
| `DS_REV_Y_2025.xlsx` | Annual revenue per firm, in thousands USD |
| `DS_MV_T_USD_Y_2025.xlsx` | Annual market cap per firm (used for carbon footprint in Part II) |
| `Risk_Free_Rate_2025.xlsx` | Monthly risk-free rate (used for Sharpe ratio) |
| `Template for Part I-SAAM (1).xlsx` | The template to fill and submit on April 12 |

**Output files produced by the notebook:**
- `SAAM_Part1_Results.xlsx` — the filled template, ready to submit
- `cumulative_returns_part1.png` — the plot to paste into the template

---

## Step-by-step walkthrough

### 1. Load the data

All the Datastream files share the same format: the first row is a broken error message from the data provider (you can ignore it), then each row is a firm indexed by its name, the first column is the ISIN code, and the rest of the columns are time periods (months or years depending on the file).

We load each file the same way and split it into the ISIN series and the time-series data.

---

### 2. Filter to AMER + EUR

Since we're Group AD, we only care about firms from North America (AMER) and Europe (EUR). The `Static_2025.xlsx` file tells us which region each firm belongs to. We keep only the ~1,300 firms that match.

---

### 3. Data cleaning

This is the most important step because the raw Datastream data has several issues.

**Delisted firms**
When a company gets delisted (goes bankrupt, gets acquired, etc.), Datastream adds `DEAD - DELIST.DD/MM/YY` to the firm's name (e.g., `CREDIT SUISSE GROUP DEAD - DELIST.14/06/23`). We need to:
- Set all prices *after* the delisting date to NaN (the firm no longer exists)
- Set the price *at* the delisting month to 0, so that the return for that month comes out as -100% — which is the correct economic outcome for an investor who held the stock to the end

**Very low prices**
For some firms the total return index drops below 0.5 (sometimes even rounds to 0). Dividing by a number close to zero gives extreme or infinite returns that are not real. We treat any RI below 0.5 as missing.

**Computing monthly returns**
Simple return: `R_t = P_t / P_{t-1} - 1`. We're careful *not* to forward-fill prices before computing returns — if a price is missing we want the return to be NaN too, not a fake zero. Returns are capped at -100% (you can't lose more than you invested).

**CO2 and revenue gaps**
Firms don't always report CO2 every year. The rule from the project: if data is missing in the middle or at the end of the sample, carry forward the last known value. If data is missing at the start (firm hadn't reported yet), leave it as NaN — you can't invest in that firm yet.

---

### 4. Investment set — who can we invest in each year?

Each year from 2013 to 2024, we decide which firms are eligible for the next year's portfolio. A firm has to pass 4 criteria:

1. **Enough return history** — at least 36 monthly observations (3 years) in the past 10-year window. You need data to estimate volatility.

2. **Valid price at year-end** — if the price is missing or below 0.5 at the end of year Y, we don't invest in it for year Y+1. We don't know its current value.

3. **Not stale** — if more than 50% of the firm's monthly returns are exactly 0, it means the stock basically doesn't trade. Including it would artificially lower the estimated portfolio variance, which would make the optimizer over-weight illiquid stocks.

4. **Has CO2 data** — even for Part I we enforce this, so that the investment universe is consistent with Part II. A firm without any CO2 data gets excluded.

**An important quirk we ran into**: Datastream stores end-of-month dates as the actual last *trading* day, not always the 31st. December 2016 is stored as Dec 30 (a Friday), December 2017 as Dec 29 (a Friday). If you hardcode `Timestamp(2016, 12, 31)` to check the price, you get nothing — that date doesn't exist in the data — and the entire investment set comes out empty for that year. The fix is to dynamically find whichever December column is in the data, rather than assuming it's always the 31st.

---

### 5. Minimum variance optimization

At the end of each year Y, for the firms in the investment set:

- Estimate the expected return vector and covariance matrix from the past 120 months (10 years) of monthly returns. Missing values within the window are filled with the cross-sectional average for that month.
- Solve: **minimize portfolio variance subject to: weights sum to 1, all weights ≥ 0** (long-only).

We use the CVXPY library with the CLARABEL solver. We initially tried scipy's generic optimizer (SLSQP) but it took 10+ minutes per year on ~850 stocks. CLARABEL solves the same problem in under half a second.

Typical result: out of 800–1,100 eligible firms, only **27–43 get a nonzero weight**. This is completely normal for min-variance — the optimizer concentrates the portfolio in a small set of low-volatility, low-correlation stocks.

---

### 6. Portfolio returns

**Min-variance**: weights are set at end of year Y, then held throughout year Y+1. The weights *drift* as prices move — we don't rebalance monthly. The drift formula is:

```
w_{i,t+1} = w_{i,t} × (1 + R_{i,t+1}) / (1 + R_portfolio,t+1)
```

This is just reflecting the fact that if a stock goes up more than the portfolio, its share of the portfolio grows naturally.

**Value-weighted**: each month, each firm's weight is its market cap at the end of the *previous* month divided by the total market cap of all AMER+EUR firms. We use previous month's cap to avoid look-ahead bias.

---

### 7. Summary statistics

| Statistic | How it's computed |
|---|---|
| Annualized average return | monthly mean × 12 |
| Annualized volatility | monthly std × √12 |
| Annualized cumulative return | geometric mean annualized: `(∏(1+Rt))^(12/T) - 1` |
| Sharpe ratio | `mean(R - Rf) / std(R) × √12` |
| Min/Max | just the minimum and maximum monthly return |

---

### 8. Results

| | Value-Weighted | Min-Variance |
|---|---|---|
| Ann. avg. return | **11.6%** | 5.8% |
| Volatility | 14.2% | **13.6%** |
| Cumulative return | 11.1% | 5.0% |
| Sharpe ratio | **0.69** | 0.30 |
| Min monthly | -13.3% | -20.1% |
| Max monthly | +12.9% | +12.2% |

The min-variance portfolio achieves its objective (lower realized volatility: 13.6% vs 14.2%) but at a significant cost in returns and Sharpe ratio. The worst monthly loss of -20.1% might seem surprising for a "low-risk" portfolio, but it's a consequence of concentration — the optimizer puts all the weight in ~30 stocks, so if one sector gets hit hard (COVID in March 2020 is the obvious candidate), the loss is larger than for the more diversified value-weighted benchmark.

---

## Do you need to write the code in one go?

No, and you really shouldn't. The notebook is designed to be run **cell by cell**, which is the whole point of Jupyter. Here's how to work through it naturally:

1. **Run the data loading cells first** and check the shapes look right (`print(ri.shape)` etc.). This is just sanity checking.

2. **Run the cleaning cells** and check how many delisted firms you find, whether the returns look reasonable (`ret.describe()` is useful here).

3. **Test the investment set function for one year** before running the full loop. E.g., call `get_eligible_firms(2013)` and check how many firms you get and whether they look right.

4. **Run the optimization loop** — this takes about 20–30 seconds total for all 12 years with CVXPY.

5. **Spot-check the returns** before filling the template. Plot a few firms, verify the cumulative return chart makes sense directionally.

The code is structured so that each section only depends on what came before it. If something looks wrong in the returns, you can go back and fix the cleaning step and re-run from there, without redoing everything. You don't need to re-run the full notebook every time — just from the cell you changed onwards.

For the **April 12 submission**, all you need is:
- `SAAM_Part1_Results.xlsx` (already filled)
- The cumulative return plot inserted into the template

The notebook itself (`SAAM_Part1.ipynb`) is only required for the **May 29 final submission**.
