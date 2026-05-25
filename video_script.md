# Video Presentation Script — SAAM Project 2026
## Group AD | AMER + EUR | Scope 1 + Scope 2
## Total duration: ~10 minutes

*Each speaker section has a suggested time. Adapt as needed.*

---

## [SPEAKER 1] — Introduction and data (0:00 – 2:00)

[Show: title slide, data table]

---

Hi everyone. We are Group AD, and today we are presenting our SAAM project. Our universe covers North American and European equities, and we use Scope 1 and Scope 2 emissions data throughout.

The project builds five portfolios over a 12-year out-of-sample period, from January 2014 to December 2025. The first two — a value-weighted benchmark and a minimum-variance portfolio — are our baselines from Part I. The other three bring in explicit carbon constraints. We will walk through all of them.

All data come from Refinitiv Datastream. We work with monthly total return indices and market capitalisation data for around 2,545 global firms, with roughly 1,300 in our AMER-EUR universe. For carbon, we use annual Scope 1 and Scope 2 emissions, combined with firm revenue and annual market capitalisation for the two carbon metrics.

One thing we had to handle carefully was delistings. Datastream flags a delisted firm by appending the delist date to its name. We found 107 of these in our universe. For each one, we set the delisting month to a -100% return and all subsequent months to missing. If you skip that step, Datastream carries forward the last available price, and when the firm eventually disappears, the return series can look misleadingly positive. That would introduce a bias into the data, so we correct for it explicitly.

---

## [SPEAKER 2] — Investment universe and Part I methodology (2:00 – 4:30)

[Show: eligibility table, cumulative return chart]

---

Let me explain how we define the eligible investment set each year. Every December, from 2013 to 2024, we apply five filters simultaneously.

A firm needs at least 36 non-missing monthly returns in the past 120 months. It needs a valid price at December of that year. The fraction of zero monthly returns must be below 50%, which screens out very illiquid stocks. It must have at least one carbon observation by that year. And it must not be delisted.

Apply all five, and the eligible universe starts at 812 firms in 2013 and grows to over 1,150 by the early 2020s. The main driver of that growth is filter F4, the carbon disclosure requirement. As sustainability reporting became more widespread, more firms entered our eligible set each year.

For the minimum-variance portfolio, each year we solve a standard long-only quadratic programme: minimise portfolio variance, subject to full investment and no short selling. We use the CVXPY library with the CLARABEL solver, and the covariance matrix is estimated from the past 120 months of returns. Any missing returns are replaced by the cross-sectional monthly mean before computing the matrix. The solution is concentrated: typically only 24 to 42 firms receive non-zero weights out of 800 to 1,150 eligible.

The value-weighted benchmark is rebalanced every month using actual Datastream market-cap data from the prior month. This matters: using a buy-and-hold drift instead would confuse the intended market-cap weighting with the passive effect of price movements, and they are not the same thing.

On the timing: weights are set at end of December and held for 12 months using a buy-and-hold drift formula. The weights from December Y apply to returns from January Y+1 through December Y+1. Getting that off-by-one check right is important — a one-month shift would misalign the entire performance record.

---

## [SPEAKER 3] — Part I results (4:30 – 5:30)

[Show: performance table, cumulative return chart]

---

Here are the Part I numbers.

Over 144 months, the value-weighted benchmark delivered 11.45% annualised return and a Sharpe ratio of 0.680. The MVP came in at 6.16% and a Sharpe of 0.365. That is a large gap.

The reason is clear in hindsight. From 2014 to 2025, the market was dominated by the US technology rally. Large-cap tech companies carry a lot of weight in the VW benchmark and delivered strong returns over this period. The MVP deliberately avoids volatile, high-correlation stocks — which includes most of the technology sector — so it missed much of that rally.

That said, the MVP does achieve what it is designed to do. It reduces annualised volatility from 14.22% to 11.98%, a meaningful reduction of over 200 basis points. Its worst monthly return of -12.73% is also slightly better than the VW's -13.20%. So the MVP does offer a smoother ride, but the return cost over this particular period was substantial.

---

## [SPEAKER 1] — Carbon metrics (5:30 – 6:30)

[Show: carbon footprint and WACI chart]

---

Moving to Part II. Before we discuss the constrained portfolios, let me explain the two carbon metrics.

The carbon footprint is the portfolio-weighted sum of each firm's emissions divided by its annual market capitalisation. The units are tonnes of CO2 per million dollars invested. It captures how much carbon you are financing per dollar invested.

The weighted-average carbon intensity, or WACI, uses revenue instead of market cap. It measures how carbon-efficient the firms are per dollar of output, independently of their market valuation. A firm with high emissions relative to its revenue has a high WACI.

Now, here is an important result. The unconstrained MVP has a higher carbon footprint than the VW benchmark, not lower. The MVP averages 149.1 tCO2/m$ invested, compared to 116.2 for the VW — about 28% higher. Its WACI of 290.0 is much further above the VW's 168.0.

This matters because it is easy to assume that a minimum-variance portfolio, which avoids large-cap tech, would incidentally end up cleaner. But the MVP concentrates on whatever stocks have low estimated variance and low correlation, and in this universe some of those stocks happen to have above-average carbon intensity. So minimum-variance investing and low-carbon investing are not the same objective. If you want to reduce carbon exposure, you need an explicit constraint.

---

## [SPEAKER 2] — Constrained portfolios (6:30 – 8:00)

[Show: constrained portfolio performance table, VW variants chart]

---

We built three carbon-constrained portfolios.

The first is the MVP with a 50% carbon constraint. For each formation year, the budget is set to half the unconstrained MVP's footprint. The result: carbon footprint drops from 149.1 to 74.5, exactly halved. WACI falls from 290.0 to 137.5. In financial terms, the annual return drops from 6.16% to 5.92%, and Sharpe falls from 0.365 to 0.348. This is a modest but expected cost: any binding constraint removes some flexibility, so financial performance is slightly lower. The constraint works as intended.

The second portfolio, and the most practically important one, is the VW50. This is a tracking-error-minimising portfolio subject to a 50% carbon footprint reduction. Annual return falls from 11.45% to 10.73% — a cost of 72 basis points. Sharpe falls from 0.680 to 0.621. In exchange, the carbon footprint halves from 116.2 to 57.7, and the WACI nearly halves from 168.0 to 85.5. For investors with ESG mandates or carbon-reporting requirements, 72 basis points per year for a 50% footprint reduction is a limited and defensible cost.

The third portfolio is the VWNZ, which uses a tightening carbon budget anchored to the 2013 VW footprint, declining at 10% per year on a compound basis. By 2024, the budget is about 28% of the 2013 baseline. Financially it is close to VW50: return 10.78%, Sharpe 0.625. But its average carbon footprint is 96.8, notably higher than VW50's 57.7.

The explanation is in the baseline design. The VW50 always targets 50% of the current benchmark footprint, which fell from around 184 tCO2/m$ in 2013 to around 60 by 2025. So in the later years, the VW50 constraint is quite tight. The VWNZ is anchored to the 2013 level and becomes relatively more relaxed as the market decarbonises on its own. The lesson is that the choice of reference point for a net-zero mandate has real long-run consequences for carbon outcomes.

---

## [SPEAKER 3] — Discussion and conclusion (8:00 – 10:00)

[Show: all-portfolios cumulative return chart, limitations slide]

---

Let me close with a few takeaways.

First, cutting the carbon footprint by 50% relative to the VW benchmark was achievable at around 72 basis points of annual return cost in this sample. That is a limited price for a clearly measurable environmental outcome. We should be careful not to generalise too much from one 12-year window, but the result is consistent with what the broader literature finds for large, diversified universes.

Second, minimum variance is not a carbon strategy. The MVP in our universe ended up with a higher carbon footprint than the VW benchmark, both in CF and WACI terms. If carbon reduction is an objective, you need an explicit constraint. The VW50 is the clean solution here.

Third, the design of the carbon constraint matters, especially over long horizons. A constraint anchored to a fixed historical baseline becomes progressively less ambitious as the market itself decarbonises. The VWNZ and VW50 look similar in financial terms, but the VW50 delivered a much lower average carbon footprint because its reference always moved with the current benchmark. Investors need to be explicit about what their constraint is anchored to.

Fourth, there are real limitations to acknowledge. We use the plain sample covariance matrix without shrinkage, which produces a concentrated MVP. Our carbon data relies on self-reported firm disclosures, which vary in quality over time and across countries. We ignore transaction costs. And we do not test performance differences statistically — with 144 monthly observations and one non-overlapping sample, formal tests would have limited power, so everything we have shown is descriptive.

Finally, we only cover Scope 1 and Scope 2 emissions. For oil and gas companies and for banks, most of the climate exposure sits in Scope 3 — downstream combustion and financed emissions. Including Scope 3 would change the picture, but data quality is not yet good enough for reliable use in an optimisation framework.

To summarise: in this sample, a 50% carbon footprint reduction cost around 72 basis points of annual return. The VW50 is the most defensible strategy for a carbon-sensitive investor. Minimum variance is not a substitute for an explicit carbon constraint, and the design of the baseline matters greatly for long-run net-zero ambition.

Thank you.

---

*Total: ~10 minutes | Group AD | SAAM Project 2026*
