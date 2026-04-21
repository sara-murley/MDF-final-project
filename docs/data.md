---
layout: default
title: Data & Code
nav_order: 3
---

# Data & Code
{: .fs-8 }

Datasets, notebooks, and instructions for reproducing this analysis
{: .fs-5 .fw-300 }

---

{: .highlight }
All code for this project is publicly available on GitHub. The analysis is structured as a series of documented Jupyter notebooks designed to be run sequentially.

## Repository Structure

<!--
FILL IN: Update this tree to match your actual repo structure.
-->

```
MDF-final-project/
├── docs/                  # This website
├── notebooks/
│   ├── 01_data_ingest.ipynb
│   ├── 02_cleaning.ipynb
│   ├── 03_eda.ipynb
│   ├── 04_analysis.ipynb
│   └── 05_visualization.ipynb
├── data/
│   ├── raw/               # Original source files (or download instructions)
│   └── processed/         # Cleaned, merged panel dataset
├── outputs/
│   └── figures/           # Exported charts and maps
├── requirements.txt
└── README.md
```

## Getting Started

### Prerequisites

<!--
FILL IN: Adjust Python/R version and package list as needed.
-->

```bash
# Clone the repository
git clone https://github.com/sara-murley/MDF-final-project.git
cd MDF-final-project

# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Running the Analysis

Run the notebooks in order:

```bash
jupyter notebook notebooks/
```

| Notebook | Description | Runtime |
|---|---|---|
| `01_data_ingest.ipynb` | Downloads and loads raw data from sources | ~[X] min |
| `02_cleaning.ipynb` | Standardizes, merges, and validates the panel | ~[X] min |
| `03_eda.ipynb` | Exploratory analysis, distributions, missing data | ~[X] min |
| `04_analysis.ipynb` | Main regression models and robustness checks | ~[X] min |
| `05_visualization.ipynb` | Generates all figures used on this site | ~[X] min |

## Data Access

<!--
FILL IN: For each dataset, either provide a direct download link, explain how to access it,
or note if it's included in the repo. Be specific — a reader should be able to exactly 
replicate your data environment.
-->

| Dataset | Access | License |
|---|---|---|
| [ITU Broadband Data] | [Download link or API instructions] | [License] |
| [World Bank WDI] | [Link — freely available via API] | CC BY 4.0 |
| [Other dataset] | [Instructions] | [License] |

{: .note }
**Data note:** [FILL IN any data use restrictions, registration requirements, or versioning notes. 
For example: "The ITU data requires a free account registration at itu.int. We use the 2023 release 
(accessed [date]). Results may differ slightly with updated releases."]

## Key Code Snippets

<!--
FILL IN: Highlight 2–3 meaningful code snippets that illustrate your core methodology.
Choose things that show analytical depth — your panel merge, your main regression, 
your instrument construction, or a key visualization. Add explanatory text around each.
-->

### Panel Construction

```python
# FILL IN: Paste the core data merge / panel construction code here
# Example structure:

import pandas as pd

# Load datasets
broadband = pd.read_csv("data/raw/itu_broadband.csv")
wdi = pd.read_csv("data/raw/wdi_indicators.csv")

# Standardize country codes
# broadband["iso3"] = ...
# wdi["iso3"] = ...

# Merge on country-year
panel = broadband.merge(wdi, on=["iso3", "year"], how="inner")
print(f"Panel dimensions: {panel.shape[0]} observations, {panel['iso3'].nunique()} countries")
```

### Main Regression

```python
# FILL IN: Paste your primary model estimation code here.
# If using linearmodels for panel FE:

# from linearmodels.panel import PanelOLS
# model = PanelOLS.from_formula(
#     "outcome ~ broadband + controls + EntityEffects + TimeEffects",
#     data=panel.set_index(["iso3", "year"])
# )
# results = model.fit(cov_type="clustered", cluster_entity=True)
# print(results.summary)
```

### [Third Key Step — e.g., Instrument Construction or Key Visualization]

```python
# FILL IN: Add a third meaningful snippet.
```

## Results Tables

<!--
FILL IN: Paste your main regression output table(s) here. You can format as markdown tables
or embed as images if you exported them. Include at minimum:
- Coefficient on broadband variable
- Standard errors (clustered)
- R-squared / within R-squared
- N observations, N countries
- Fixed effects specification
-->

### Table 1: Main Results

| | (1) Baseline | (2) + Controls | (3) FE | (4) Preferred |
|---|---|---|---|---|
| Broadband penetration | | | | |
| | *(.s.e.)* | *(.s.e.)* | *(.s.e.)* | *(.s.e.)* |
| Controls | No | Yes | Yes | Yes |
| Country FE | No | No | Yes | Yes |
| Year FE | No | No | Yes | Yes |
| N | | | | |
| Countries | | | | |

<!--
FILL IN: Replace placeholder cells with your actual estimates.
Indicate significance with * p<0.10, ** p<0.05, *** p<0.01
-->

[View Visualizations →](explorer){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[GitHub Repository →](https://github.com/sara-murley/MDF-final-project){: .btn .fs-5 .mb-4 .mb-md-0 }