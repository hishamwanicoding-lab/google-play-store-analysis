# Google Play Store Analysis — Project README

## Project Pipeline
This project follows a six-stage analysis pipeline. Each stage is its own notebook:

| Stage | Notebook | Status |
|---|---|---|
| 1. Data Understanding | `Data_Understanding.ipynb` | Complete |
| 2. Data Cleaning | `Data_Cleaning.ipynb` | Complete |
| 3. Exploratory Data Analysis | `Exploratory_Data_Analysis.ipynb` | Complete |
| 4. Data Visualization | `_Data_Visualization.ipynb` | **Stub only** — one empty code cell |
| 5. Business Insights | `_Business_Insights.ipynb` | **Empty file** — 0 bytes, no cells |
| 6. Recommendations | `Recomendations.ipynb` | **Stub only** — one empty markdown cell |

Stages 1–3 are done and reviewed. Stages 4–6 haven't been built out yet — this repo's
`Insights_and_Report.md` and `Business_Recommendations.md` (in this same output set) are
written to stand in for stages 5–6 until those notebooks are filled in.

## Data Sources
| File | Description |
|---|---|
| `googleplaystore.csv` | Raw Play Store app data (used only in Understanding/Cleaning) |
| `googleplaystore_user_reviews.csv` | Raw user review data |
| `processedplaystoredata.csv` | Cleaned app-level data (output of `Data_Cleaning.ipynb`, input to EDA) |
| `processeduserreview.csv` | Cleaned review data (output of `Data_Cleaning.ipynb`, input to EDA) |

## Stage 1 — Data Understanding
Basic profiling of both raw files: shape, size, dtypes, `describe()`, duplicate counts, and
missing-value counts. Confirms two files exist (app metadata, user reviews) and that the app
dataset has ~10,841 rows with known messy columns (Size, Installs, Price stored as text with
symbols).

## Stage 2 — Data Cleaning
Cleans each problematic column individually: strips text, converts `Size`/`Installs`/`Price`
from strings to numeric, fixes the known corrupted row (a Category-shifted record that also
produces an out-of-range `Rating` of 19.0 and a `Price` of "Everyone"), fills missing values
column-by-column (mostly with median for numeric, mode/"Unknown" for categorical), converts
`Last Updated` to datetime, and writes the result to `processedplaystoredata.csv`.

**A bug worth flagging here, because it explains the dedup issue found in the EDA review:**
the notebook computes duplicate-row removal (`playstoredata.drop_duplicates()`) and assigns it
to `processedpscopy`, but the very next cell recomputes `processedpscopy` from the *original*
`playstoredata` again (`playstoredata.dropna(how='all')`), silently discarding the duplicate
removal. So the "cleaned" file that EDA reads from was **never actually deduplicated** — this
is the root cause of the inconsistent counts flagged in the EDA review, not something that
originated in the EDA notebook itself.

There's also a no-op rename: `columns.str.replace("Android ver", ...)` doesn't match the real
column name `"Android Ver"` (capital V) — the intended rename to "Android Version Required"
silently fails.

## Stage 3 — Exploratory Data Analysis
Univariate analysis of every column, sentiment analysis on the review file, and ~20
business-question-driven bivariate analyses (category performance, pricing, ratings,
installs, update recency). See `KPI.md` and `Insights_and_Report.md` for the extracted
findings, and the earlier review notes for code-quality issues (duplicate section, one
mislabeled chart, missing `plt.show()` calls, Pearson correlation on skewed data).

## Stages 4–6 — Not Yet Built
- **Data Visualization**: currently one markdown header ("VISUALIZING CATEGORIES") and an
  empty code cell. Intended to likely hold polished/presentation-ready charts distinct from
  the analysis-stage plots in the EDA notebook.
- **Business Insights**: file is completely empty (0 bytes) — needs to be created.
- **Recommendations**: one empty markdown cell.

## Recommended Fix Order
1. Fix the duplicate-removal bug in `Data_Cleaning.ipynb` (don't overwrite `processedpscopy`
   with a fresh `dropna` off the original frame) and re-save `processedplaystoredata.csv`.
2. Re-run `Exploratory_Data_Analysis.ipynb` against the corrected file and reconcile the
   category-level numbers.
3. Fill in `_Data_Visualization.ipynb`, `_Business_Insights.ipynb`, and `Recomendations.ipynb`
   — `Insights_and_Report.md` and `Business_Recommendations.md` can seed both of the latter two.

## How to Reproduce
```bash
pip install pandas numpy matplotlib
jupyter notebook
```
Run notebooks in order: Data_Understanding → Data_Cleaning → Exploratory_Data_Analysis →
Data_Visualization → Business_Insights → Recommendations. File paths assume a
`DATA/RawData/` folder one level above the notebooks.

## Related Files
- `KPI.md` — key metrics extracted from the EDA
- `Insights_and_Report.md` — full narrative findings from the EDA
- `Business_Recommendations.md` — strategic recommendations for stakeholders
