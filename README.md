# Happiness vs. YouTube: Country-Level Analysis.”

Country-level analysis of **YouTube reach (February 2024)** and **World Happiness Report 2024**. The project answers three stacked questions: which Latin American markets are largest versus least saturated, whether YouTube penetration co-moves with national happiness, and which countries combine high well-being and income with relatively low YouTube saturation (a **Digital Opportunity Index**).

Work is done in **Jupyter notebooks** under WSL (Ubuntu), using a Python 3.12 virtual environment.

## Data

| File | Role |
| --- | --- |
| `datasource/youtube-users-by-country-2025.csv` | YouTube users and penetration by country (February 2024 columns used for 2024) |
| `datasource/World-happiness-report-2024.csv` | Ladder (happiness) score, regional indicator, **log GDP per capita**, and related WHR variables |

Join key: country **name** (`country` after rename). Happiness has no ISO code. An inner join yields **80** matched countries; **Bahrain** is dropped in Advanced because log GDP is missing (**n = 79**).

Exact-name mismatches (not joined) include pairs such as Czech Republic / Czechia, Taiwan / Taiwan Province of China, Turkey / Türkiye, and Hong Kong / Hong Kong S.A.R. of China. Oman, Qatar, and Puerto Rico appear in YouTube but not in the happiness file.

## Stack

- Python 3.12, `pandas`, `matplotlib`, `seaborn`, `scikit-learn` (`MinMaxScaler`)
- Jupyter / `ipykernel`
- Dependencies: `requirements.txt`

## Layout

```text
ChallengeDT/
├── datasource/           # raw CSVs (do not overwrite)
├── notebooks/
│   ├── 01-begginer.ipynb
│   ├── 02-intermediate.ipynb
│   └── 03-advanced.ipynb
├── outputs/
│   ├── figures/          # PNG charts
│   └── tables/           # Top 10 opportunity CSV
├── src/                  # reserved for reusable modules
├── requirements.txt
└── README.md
```

## Stages

### 1. Beginner — LATAM YouTube market (2024)

**Question.** Which LATAM countries have the most YouTube users, and which look less saturated?

**Method.** YouTube file only. Explicit LATAM country list (exact CSV spellings), `isin` filter, set-difference diagnostic for names in the list that are missing from the file. Puerto Rico is excluded as a territory. Two horizontal bar charts on the same 17 countries: users (millions) and penetration (%).

**Outputs.** `outputs/figures/01_latam_users_2024.png`, `outputs/figures/02_latam_penetration_2024.png`.

**Reading.** Volume and saturation do not rank the same way. Brazil leads in users. Chile leads in penetration. Peru is relatively large but less saturated than the other large markets. Uruguay and Panama are small and already highly penetrated.

### 2. Intermediate — penetration vs happiness

**Question.** Are countries with higher YouTube penetration also happier?

**Method.** Inner join on `country`. Document unmatched names on both sides. Pearson and Spearman correlation between `penetration_2024` and `happiness` (Ladder score). Scatter plot with a linear fit, **n = 80**.

**Outputs.** `outputs/figures/03_penetration_vs_happiness.png`.

**Reading.** Moderate **positive** association (Pearson \(r \approx 0.53\), Spearman \(\rho \approx 0.57\)). The methods agree. The cloud rises to the right but is spread out. This is a **country-level** association, not evidence that YouTube causes happiness. Income and internet access can sit behind both series. At least one country combines very high penetration with very low happiness.

### 3. Advanced — Digital Opportunity Index

**Question.** Which matched countries look like high capacity (happiness + income) and relatively low YouTube saturation?

**Method.** Same inner join, drop missing log GDP (**n = 79**). `MinMaxScaler` on `happiness`, `log_gdp`, and `penetration_2024` **fitted on this sample**. Index:

`opportunity_index = happiness_norm + log_gdp_norm − penetration_norm`

User counts are **not** in the formula; they only set bubble **size**. GDP is **log GDP per capita**, not raw GDP. Weights are those of the brief (equal capacity terms, penetration subtracted).

**Outputs.** `outputs/figures/04_opportunity_bubbles.png` (X = log GDP, Y = happiness, size = users, colour = penetration), `outputs/tables/top10_opportunity.csv`.

**Reading.** Many Top 10 names are rich, high-happiness, already quite penetrated (Nordics, United States, Switzerland, and similar). They score well because capacity is extreme and penetration is not always the sample maximum. The **Philippines** is the distinct Top 10 case: mid-range capacity, much **lower** penetration. On the bubble chart, opportunity is **upper-right but not fully yellow** (yellow = high saturation).

## How to run

From the project root in WSL, with `.venv` active:

```bash
source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter lab
```

Open the notebooks in order (`01` → `02` → `03`). Select the kernel that points at `ChallengeDT/.venv/bin/python`.

## Limits (all stages)

- LATAM membership is an explicit list, not a column in the YouTube file.
- Joins are exact string matches on English country names.
- Penetration is a percentage proxy, not a roster of non-users.
- The opportunity index is a constructed score, not a causal or predictive model.
