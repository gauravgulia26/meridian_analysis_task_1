# Meridian Spend Analysis & Should-Cost Modeling

This repository contains the deliverables for the interview assignment on spend data cleaning, categorization, and should-cost benchmarking for Meridian Auto Components.

---

## 🧭 Reviewer Guide & Quick Navigation

For quick review, the core findings and analysis are organized by level of detail:

1. **Executive Summary & Key Takeaways**:
   - [`reports/summary.pdf`](reports/summary.pdf) (or [`references/summary.md`](references/summary.md)) — High-level summary of findings, savings opportunities, and spend risks.
2. **Detailed Methodology & Analysis**:
   - [`reports/eda.pdf`](reports/eda.pdf) (or [`references/eda.md`](references/eda.md)) — Data quality audit, cleaning rules, and category classification logic.
   - [`reports/methodology.pdf`](reports/methodology.pdf) (or [`references/methodology.md`](references/methodology.md)) — Mathematical formulation, assumptions, and should-cost breakdown.
3. **Execution Notebooks (Code & Analysis)**:
   - [`notebooks/`](notebooks/) — End-to-end data processing and modeling workflow (executed in numerical order 1 → 4).

---

## 📁 Repository Structure

```text
├── reports/                 # Final PDF deliverables for review
│   ├── summary.pdf          # Executive summary: key savings & risks
│   ├── eda.pdf              # Track A: Data quality & spend classification
│   └── methodology.pdf      # Track B: Should-cost calculation methodology
│
├── references/              # Markdown sources for the reports
│   ├── summary.md           # Summary documentation
│   ├── eda.md               # Detailed EDA and data-cleaning audit
│   └── methodology.md       # Comprehensive should-cost model documentation
│
├── notebooks/               # Step-by-step analysis notebooks
│   ├── 1-intitial-EDA.ipynb # Initial data profiling and missing value audit
│   ├── 2-EDA.ipynb          # Text parsing, category reclassification & cleaning
│   ├── 3-Should-Cost.ipynb  # Preliminary conversion cost logic & material rates
│   └── 4-Should_Cost.ipynb  # Final should-cost model & scenario calculations
│
├── data/                    # Datasets
│   ├── meridian_spend_data.csv  # Original raw dataset
│   └── interim/             # Versioned datasets at each processing stage (v1–v5)
│
└── figures/                 # Exported visualizations and plots
    └── Condensed_Categories.png # Category consolidation mapping chart
```

---

## 📓 Notebook Pipeline Overview

If reviewing the code, the notebooks follow a sequential progression:

1. **`1-intitial-EDA.ipynb`**: Imports the raw dataset, conducts preliminary sanity checks on schemas/types, identifies missing values, and exports initial baseline data (`cleaned_data.csv`).
2. **`2-EDA.ipynb`**: Performs deep-dive text parsing on part descriptions, corrects misclassifications and typos (e.g., *Farsteners*), and groups parts into consolidated manufacturing categories (`cleaned_data_v3.csv`).
3. **`3-Should-Cost.ipynb`**: Defines material pricing ranges and establishes initial conversion cost driver functions (`cleaned_data_v4.csv`).
4. **`4-Should_Cost.ipynb`**: Implements baseline complexity metrics, secondary operations logic, and computes final Low / Base / High should-cost benchmarks and supplier price gaps (`cleaned_data_v5.csv`).
