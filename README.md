# 🔧 Automotive Repair Data Analysis & Insights

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?logo=pandas&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--Learn-1.x-F7931E?logo=scikit-learn&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-Visualization-4C72B0)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![License](https://img.shields.io/badge/License-MIT-green)

> End-to-end data validation, cleaning, integration, and exploratory analysis of automotive warranty repair records — surfacing cost drivers, failure patterns, and plant-level quality gaps for stakeholder decision-making.

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Dataset Description](#-dataset-description)
- [Tasks Performed](#-tasks-performed)
- [Key Findings](#-key-findings)
- [Visualisations](#-visualisations)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [How to Run](#-how-to-run)
- [Actionable Recommendations](#-actionable-recommendations)

---

## 📖 Project Overview

This project analyses **automotive warranty repair records** across two datasets — a transactional repair log and a reference/metadata layer — to identify failure trends, high-cost components, and plant-level quality issues.

The analysis covers **3 structured tasks**:

| Task | Focus | Output |
|------|-------|--------|
| Task 1 | Data Validation & Cleaning | Cleaned + tagged CSV |
| Task 2 | Data Preparation & Integration | Merged dataset CSV |
| Task 3 | Exploratory Data Analysis (EDA) | Trend charts + stakeholder summary |

**Business question answered:** *Which components, plants, and failure modes are driving repair costs — and what should stakeholders act on first?*

---

## 📂 Dataset Description

### Dataset 1 — Transactional Repair Log (`SA - Data for Task 1.xlsx`)
| Property | Detail |
|---|---|
| Rows | 100 |
| Columns | 52 (cleaned to 51 after dropping high-null column) |
| Key fields | `VIN`, `TRANSACTION_ID`, `REPAIR_DATE`, `CORRECTION_VERBATIM`, `CUSTOMER_VERBATIM`, `GLOBAL_LABOR_CODE_DESCRIPTION`, `TOTALCOST`, `LBRCOST`, `PLANT` |
| Free text fields | `CORRECTION_VERBATIM`, `CUSTOMER_VERBATIM` — rich with DTC codes, part names, technician actions |
| Null columns | `CAMPAIGN_NBR` (>80% null → dropped), `CAUSAL_PART_NM` (5 nulls → imputed) |

### Dataset 2 — Reference / Metadata Layer (`SA - Data for Task 2.xlsx`)
| Property | Detail |
|---|---|
| Rows | 500 |
| Columns | 15 |
| Key fields | `PRIMARY_KEY`, `ORDER_NO`, `MANUFACTURER`, `MODEL`, `PRODUCT_CATEGORY`, `FAILURE_CONDITION` |
| Join challenge | No direct shared column with Dataset 1 — required semantic alignment |

---

## ✅ Tasks Performed

### Task 1 — Data Validation & Cleaning

- **Column normalisation:** Stripped whitespace, lowercased all column names, replaced spaces with underscores for consistency
- **Null handling:**
  - Categorical nulls → filled with `"Unknown"`
  - Float/numeric nulls → filled with `0.0`
  - `campaign_nbr` column (>80% null) → dropped entirely
- **String cleaning:** Stripped leading/trailing whitespace from all object columns
- **Outlier review:** Reviewed `totalcost` and `lbrcost` distributions
- **Free-text tag extraction:** Generated structured tags from `global_labor_code_description`:
  - `component_tag` — e.g. `steering_wheel`, `steering_wheel_module`, `horn_switch`, `wiring_harness`, `spoke_cover`
  - `action_tag` — e.g. `replacement`, `repair`

**Top 5 Critical Columns Identified:**

| # | Column | Why It Matters |
|---|--------|----------------|
| 1 | `component_tag` | Direct proxy for failure type — drives repair action |
| 2 | `totalcost` | Primary financial KPI for stakeholders |
| 3 | `lbrcost` | Signals repair complexity and technician effort |
| 4 | `plant` | Reveals geographic/process quality variation |
| 5 | `repair_date` | Enables temporal trend analysis |

---

### Task 2 — Data Preparation & Integration

- **Primary key challenge:** `set(df1.columns).intersection(set(df2.columns))` returned an empty set — no direct shared key existed between datasets
- **Solution:** Semantic alignment — renamed `global_labor_code_description` (Dataset 1) and `failure_condition_-_failure_component` (Dataset 2) to a unified `labor_description` column, then merged on this proxy key
- **Join type:** `LEFT JOIN` — chosen to preserve all 100 repair events from the primary dataset, even where reference metadata was unavailable
- **Result:** 100-row merged dataset with 67 columns (`task2_merged.csv`)

> **Note on join implications:** An `INNER JOIN` would have dropped repair records with no matching reference entry, losing potentially important failure events. A `FULL OUTER JOIN` would have introduced 400 additional reference-only rows with no transactional value. Left join was the correct choice for this use case.

---

### Task 3 — Exploratory Data Analysis

Three key visualisations produced on the merged dataset:

1. **Total Cost by Component** — which components are most expensive to repair
2. **Labor Cost by Component** — which repairs are most complex/time-intensive
3. **Failure Count by Plant** — which manufacturing plants have the highest defect rates

> See [Visualisations](#-visualisations) section below for findings.

---

## 📊 Key Findings

### 1. Component Cost Distribution
- **Steering wheel assembly** components (module, full assembly) account for the majority of total repair costs
- A small subset of component types drives a disproportionately large share of total spend — classic Pareto (80/20) pattern

### 2. Labor Cost vs. Total Cost
- Components with high **labor cost** relative to total cost indicate complex, time-intensive repairs — flagging candidates for design simplification or technician training
- Steering wheel module replacements show the highest labor-to-part-cost ratio

### 3. Plant-Level Failure Rates
- Significant variation exists across manufacturing plants in failure count
- Top-failure plants should be prioritised for process audits and quality improvement programmes

### 4. Tag Distribution
- `replacement` dominates as the primary action tag — very few `repair` actions exist
- This suggests limited opportunity for partial fixes; design improvements or preventive maintenance could reduce full replacement rates

### 5. Free-text Insights
- `CORRECTION_VERBATIM` fields contain DTC codes (e.g. `U0229`, `U1530`), TAC case references, and specific part numbers — a rich source for further NLP-based root cause categorisation

---

## 📈 Visualisations

All charts are generated in-notebook using Matplotlib and Seaborn.

| Chart | Key Insight |
|-------|-------------|
| Failure Count by Component (bar) | Steering wheel is the highest-frequency failure |
| Total Cost by Component (bar) | Steering wheel module dominates cost |
| Labor Cost by Component (bar) | Module repairs are most labor-intensive |
| Failure Count by Plant (bar) | Plant-level quality variance is significant |

> To regenerate all visualisations, run the notebook end-to-end (see [How to Run](#-how-to-run)).

---

## 🛠 Tech Stack

```
Language      : Python 3.10+
Data wrangling: Pandas, NumPy
ML utilities  : Scikit-learn (preprocessing utilities)
Visualisation : Matplotlib, Seaborn
NLP / tagging : Regex, keyword extraction (custom functions)
Environment   : Google Colab / Jupyter Notebook
Output formats: CSV (cleaned & merged datasets)
```

---

## 📁 Project Structure

```
automotive-data-analysis/
│
├── data/
│   ├── SA - Data for Task 1.xlsx          # Raw transactional repair dataset
│   └── SA - Data for Task 2.xlsx          # Raw reference/metadata dataset
│
├── outputs/
│   ├── task1_cleaned_and_tagged.csv       # Cleaned + tagged Task 1 output
│   └── task3_cleaned_and_tagged.csv       # Merged dataset for EDA
│
├── Assessment_Data_Analysis_Insights.ipynb  # Main analysis notebook
├── README.md                                # This file
└── requirements.txt                         # Python dependencies
```

---

## ▶️ How to Run

### Option A — Google Colab (recommended)
1. Upload both `.xlsx` data files to your Colab `/content/` directory
2. Open `Assessment_Data_Analysis_Insights.ipynb` in Google Colab
3. Run all cells (`Runtime → Run all`)

### Option B — Local Jupyter
```bash
# 1. Clone the repository
git clone https://github.com/LabsVelns/automotive-data-analysis.git
cd automotive-data-analysis

# 2. Install dependencies
pip install -r requirements.txt

# 3. Launch Jupyter
jupyter notebook Assessment_Data_Analysis_Insights.ipynb
```

### `requirements.txt`
```
pandas>=2.0.0
numpy>=1.24.0
matplotlib>=3.7.0
seaborn>=0.12.0
scikit-learn>=1.3.0
openpyxl>=3.1.0
jupyter>=1.0.0
```

> **Note:** Update the file paths in Cell 3 and Cell 25 from `/content/SA - Data for Task X.xlsx` to your local path if running outside Colab.

---

## 💡 Actionable Recommendations

Based on the analysis, three priority actions are recommended for stakeholders:

### 1. 🎯 Prioritise High-Impact Components
Steering wheel assemblies and modules appear in both the **highest failure frequency** and **highest total cost** buckets. Recommend:
- Escalate to supplier for design review
- Initiate quality audit on incoming parts batch
- Evaluate whether a Technical Service Bulletin (TSB) is warranted

### 2. 🔁 Investigate Replacement-Only Patterns
The near-absence of `repair` action tags (vs. `replacement`) indicates that partial fixes are not being attempted. Recommend:
- Evaluate whether repairability can be engineered into the steering wheel module
- Introduce technician training for diagnostic-first workflows before full replacement

### 3. 🏭 Conduct Plant-Level Process Audits
Variation in failure counts across plants suggests differences in assembly conditions, tooling, or QC processes. Recommend:
- Share best practices from low-failure plants across facilities
- Initiate targeted audits at the top-2 failure-rate plants
- Add plant as a stratification variable in future warranty analytics

---

## 👤 Author

**Harshad Mane**
ML Engineer | Data Science | AI/ML Production Systems

[![LinkedIn](https://img.shields.io/badge/LinkedIn-harshad0074-0A66C2?logo=linkedin&logoColor=white)](https://linkedin.com/in/harshad0074)
[![GitHub](https://img.shields.io/badge/GitHub-LabsVelns-181717?logo=github&logoColor=white)](https://github.com/LabsVelns)
[![Email](https://img.shields.io/badge/Email-harshadmane1293@gmail.com-D14836?logo=gmail&logoColor=white)](mailto:harshadmane1293@gmail.com)

---
