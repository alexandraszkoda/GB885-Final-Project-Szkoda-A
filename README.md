# RUSH Sportswear — US Sales Analysis (2020–2021)

Exploratory analysis of RUSH's raw US sales data for the VP of US Sales, covering FY2020 and FY2021.
Built for GB885 Final Project.

**Deliverable:** [`notebooks/RUSH_sales_analysis.ipynb`](notebooks/RUSH_sales_analysis.ipynb) —
end-to-end acquisition, inspection, cleaning, business-question answers, and independent insights.

---

## Answers to the VP's questions

| # | Question | Answer |
|---|---|---|
| 1 | Product category with the highest sales in 2021 | **Men's Street Footwear — $22,690,427** (23.7% of 2021 revenue) |
| 2 | State with the highest women's-product sales in 2021 | **Maine — $2,176,301** |
| 3 | State with the highest men's-product sales in 2021 | **Delaware — $2,334,300** |
| 4 | Retailer purchasing the most units in 2021 | **Foot Locker — 1,097,410 units** (54.4% of units) |
| 4 | Retailer purchasing the most units in 2020 | **Amazon — 317,930 units** (68.8% of units) |

## Headline insights

1. **Revenue grew 4x to $95.9M, but 78% of the growth came from new distribution** — Foot Locker and
   Walmart had no 2020 activity and delivered $55.9M in 2021.
2. **Amazon fell 40%** ($17.5M → $10.5M) — the only account in decline, and the largest account in 2020.
3. **Online is now the largest *and* the highest-margin channel** — 42% of revenue at a 46.4%
   operating margin, versus 39.5% Outlet and 35.6% In-store.
4. **2020 seasonality was distorted by COVID** (April peak, June trough); 2021 shows a normal
   July/December retail rhythm and should be the planning baseline.
5. **Concentration risk:** Foot Locker alone is 56% of 2021 revenue.

---

## Repository structure

```
.
├── data/                            # Raw source extracts (read-only inputs)
│   ├── TABLE_SALES_885.csv
│   ├── TABLE_RETAILER_885.csv
│   └── TABLE_PRODUCTS_885.csv
├── notebooks/
│   └── RUSH_sales_analysis.ipynb    # Main analysis notebook
├── figures/                         # Charts exported by the notebook (.png)
├── outputs/
│   └── rush_clean_orders.csv        # Cleaned, merged order-level dataset
├── presentation/
│   ├── RUSH_2021_Sales_Review.pptx  # Slides for the 3-5 minute VP presentation
│   └── presentation_script.md       # Timed speaking script
├── requirements.txt
├── .gitignore
└── README.md
```

## How to run

```bash
git clone <this-repo-url>
cd rush-sales-analysis
pip install -r requirements.txt
jupyter notebook notebooks/RUSH_sales_analysis.ipynb
```

Run all cells top to bottom. The notebook resolves `data/` whether it is launched from the repo root
or from `notebooks/`, and writes charts to `figures/` and the cleaned dataset to `outputs/`.

**Google Colab:** upload the three CSVs, then replace the `DATA_DIR` line in Section 4 with:

```python
from google.colab import files; files.upload()
DATA_DIR = Path(".")
```

---

## Data

Three raw tables, joined on `RETAILER_ID` and `PRODUCT_ID`:

| Table | Rows | Grain |
|---|---|---|
| `TABLE_SALES` | 9,648 | One order line |
| `TABLE_RETAILER` | 110 | One retailer–location |
| `TABLE_PRODUCTS` | 6 | One product category |

Revenue is **derived**, not stored: `TOTAL_SALES = PRICE_PER_UNIT × UNITS_SOLD`, and
`OPERATING_PROFIT = TOTAL_SALES × OPERATING_MARGIN`.

### Data quality issues found and how they were handled

| # | Issue | Rows | Treatment |
|---|---|---|---|
| 1 | `SALES_METHOD` misspelled `Ootlet` | 20 | Recoded to `Outlet` |
| 2 | `UNITS_SOLD` contains the string `***` | 2 | Converted to missing, then imputed |
| 3 | `PRICE_PER_UNIT` missing | 2 | Imputed |
| 4 | `PRICE_PER_UNIT` = `99999` (sentinel) | 1 | Treated as missing, then imputed |
| 5 | `UNITS_SOLD` = 0 on priced orders | 4 | Retained and flagged (contribute $0) |
| 6 | Orphan `RETAILER_ID` `999999999` | 1 | Retained; location shown as *Unknown* |
| 7 | Undocumented `MONTH`/`DAY`/`YEAR` columns | all | Verified against `INVOICE_DATE`, then dropped |
| 8 | `RETAILER_ID` not unique — 4 collisions | 8 | Dimension de-duplicated before the join |
| 9 | `TABLE_PRODUCTS` pipe-delimited + BOM | file | Handled at import |

Imputation rule: median of the same product, at the same retailer location, in the same year, with a
product-level median fallback. Five values out of 28,944 measures (0.02%) were imputed.

**Issue #4 was the costly one.** One order of 520 units priced at $99,999 would have added **$52M of
phantom revenue** to a $96M year — a 54% overstatement — and would have changed the answer to two of
the VP's four questions.

**Issue #8 is a system design flaw worth reporting upstream.** `RETAILER_ID` is constructed as
`[retailer initial] + 00 + [region initial] + [state first 2] + [city first 2]`, which collides
whenever two retailers share a first letter (Walmart/West Gear) or two places share letter pairs
(New Jersey/New York, Newark/New York). Section 12 of the notebook re-runs Q4 under the opposite
attribution and confirms the answer does not change.

---

## Tools

Python · pandas · NumPy · Matplotlib
