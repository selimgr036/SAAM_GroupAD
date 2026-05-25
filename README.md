# SAAM Project — Group AD
### MSc Finance 2026 | North America + Europe | Scope 1 + Scope 2

---

## How to run the notebook (VS Code)

### Step 1 — Open the project folder in VS Code

File → Open Folder → select the `SAAM Project` folder.

> Make sure you open the **folder**, not just the file. The notebook uses relative paths like `Data/...` and `outputs/...`, so it must run from the project root.

### Step 2 — Open the notebook

Click `SAAM_Final.ipynb` in the Explorer panel on the left.

### Step 3 — Select the Python kernel

At the top-right of the notebook, click **Select Kernel** → choose the Python interpreter that has the dependencies installed (usually the one labelled `Python 3.x.x` or your virtual environment name).

If you are unsure which interpreter to use, open the VS Code terminal (`Ctrl+`` `) and run:
```bash
python3 -c "import numpy, pandas, cvxpy, openpyxl, matplotlib; print('all ok')"
```
If it prints `all ok`, that interpreter is the right one.

### Step 4 — Run everything

Click the **↓↓ Run All** button at the top of the notebook.

Or use the command palette: `Cmd+Shift+P` → type **"Run All Cells"** → Enter.

**That's it.** The notebook reads all data from `Data/`, runs all calculations, and saves every output automatically.

---

## What happens during the run

The notebook runs 12 sections in order:

| Section | What it does | Approx. time |
|---|---|---|
| 1 — Imports & paths | Loads libraries, sets up `outputs/` folders | < 1 s |
| 2 — Load data | Reads all 9 Excel files from `Data/` | 15–30 s |
| 3 — Clean returns | Applies RI filter, computes monthly returns | 5 s |
| 4 — Annual data | Forward-fills emissions, revenue, market cap | 2 s |
| 5 — Eligible universe | Runs eligibility filters for 2013–2024 | 5 s |
| 6 — Build VW + MVP | Solves 12 minimum-variance QPs, builds VW | 1–3 min |
| 7 — Part I stats | Prints performance table, saves Part I figures | 10 s |
| 8 — Carbon metrics | Computes CF and WACI for VW and MVP | 5 s |
| 9 — Constrained portfolios | Solves MVP50, VW50, VWNZ (36 QPs total) | 3–8 min |
| 10 — Carbon: all portfolios | Computes CF/WACI for all five portfolios | 5 s |
| 11 — Figures + summary | Saves all charts, prints final tables | 10 s |
| 12 — Sanity checks | Verifies 144 obs, weights, carbon constraints | 5 s |

**Total expected runtime: 5–15 minutes.**

Section 12 at the end should print:
```
=== Sanity Check Results ===
  All checks passed.
```

If it prints errors, see the Troubleshooting section below.

**Important:** After a successful run, the CSV files in `outputs/tables/` are the authoritative source of truth for all reported numbers. The report, sales pitch, and video script are all based on those outputs.

---

## Outputs

After running, you will find:

**Figures** (`outputs/figures/`):

| File | Description |
|---|---|
| `cumulative_returns_part1.png` | VW vs MVP — Figure 1 in the report |
| `carbon_metrics_evolution.png` | CF and WACI over time — Figure 2 in the report |
| `cum_all_portfolios.png` | All five portfolios — Figure 3 in the report |
| `cum_mvp_vs_mvp50.png` | MVP vs MVP50 comparison |
| `cum_vw_variants.png` | VW, VW50, VWNZ comparison |
| `cum_mvp.png` / `cum_vw.png` | Individual charts for the Excel template |

**Tables** (`outputs/tables/`):

| File | Description |
|---|---|
| `performance_summary.csv` | Full performance table (all 5 portfolios) |
| `carbon_footprint_summary.csv` | Average CF by portfolio |
| `waci_summary.csv` | Average WACI by portfolio |

**Excel** (project root):

| File | Description |
|---|---|
| `SAAM_Part1_Results.xlsx` | Part I template filled with VW and MVP returns + charts |

> The Excel file is only generated if `Archive/Template for Part I-SAAM (1).xlsx` exists. If it is missing, the notebook prints a note and continues — it does not crash.

---

## Data files (all must be in `Data/`)

| File | What it contains |
|---|---|
| `Static_2025.xlsx` | 2,545 firms: ISIN, name, country, region |
| `DS_RI_T_USD_M_2025.xlsx` | Monthly total return index, USD |
| `DS_MV_T_USD_M_2025.xlsx` | Monthly market cap, million USD |
| `DS_MV_T_USD_Y_2025.xlsx` | Annual market cap, million USD |
| `DS_CO2_SCOPE_1_Y_2025.xlsx` | Annual Scope 1 CO₂ emissions, tCO₂ |
| `DS_CO2_SCOPE_2_Y_2025.xlsx` | Annual Scope 2 CO₂ emissions, tCO₂ |
| `DS_REV_Y_2025.xlsx` | Annual revenue, thousand USD |
| `Risk_Free_Rate_2025.xlsx` | Monthly Fama-French risk-free rate |

All 9 files are already in `Data/`. **Do not move, rename, or manually edit them.**

---

## Dependencies

Everything is already installed. If you ever need to reinstall:

```bash
pip install numpy pandas cvxpy openpyxl matplotlib
```

- Python 3.9 or later required
- CVXPY version 1.3+ (includes the CLARABEL solver — no separate install needed)

---

## Project files

| File | What it is |
|---|---|
| `SAAM_Final.ipynb` | The notebook — **the only file you run** |
| `report_draft.md` | Academic report (convert to PDF to submit) |
| `sales_pitch.md` | One-page client pitch (convert to PDF to submit) |
| `video_script.md` | Script for the 10-minute video presentation |
| `methodology_final.md` | Detailed methodology notes |

---

## Converting report and sales pitch to PDF

**Option A — VS Code (easiest since you're already there):**
1. Install the **"Markdown PDF"** extension (search in Extensions panel: `Ctrl+Shift+X`)
2. Open `report_draft.md`
3. Right-click anywhere in the file → **"Markdown PDF: Export (pdf)"**
4. Repeat for `sales_pitch.md`

**Option B — Pandoc (command line):**
```bash
# Install once
brew install pandoc

# Convert
pandoc report_draft.md  -o Report_GroupAD.pdf     --pdf-engine=pdflatex
pandoc sales_pitch.md   -o SalesPitch_GroupAD.pdf --pdf-engine=pdflatex
```

**Option C — Typora:**  
Open the file → File → Export → PDF

---

## Troubleshooting

**"Module not found" error at the start:**
```bash
pip install numpy pandas cvxpy openpyxl matplotlib
```

**"FileNotFoundError: Data/..." error:**
Make sure all 9 Excel files are inside the `Data/` subfolder and that you launched Jupyter from the `SAAM Project` folder (not a subfolder).

**Solver says "infeasible" for one year:**
This is expected behaviour. The notebook falls back to unconstrained MVP or VW weights for that year and prints a warning. Results are still valid.

**Section 12 reports a sanity check failure:**
Take a screenshot of the error message and fix before submitting.

**Notebook is very slow (> 30 min):**
This should not happen. If it does, it is likely stuck on a CVXPY solve. Click the **⏹ Interrupt** button at the top of the notebook, then click **↓↓ Run All** again.

---

*Group AD | SAAM 2026 | Data: Refinitiv Datastream, 2025 vintage*
